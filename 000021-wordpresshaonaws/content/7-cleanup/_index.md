---
title : "Clean up resources"
weight : 7
chapter : false
pre : " <b> 7. </b> "
---

Delete the resources in this order. Some resources must be fully deleted before the next ones can be removed, so the commands are split into three steps. Run all of them in **AWS CloudShell**.

#### Step 1 – Disable CloudFront, delete the web tier and the databases

```bash
# 1. CloudFront: disable the distribution
CFID=$(aws cloudfront list-distributions --query "DistributionList.Items[?Comment=='WordPress on AWS (FCJ)'].Id" --output text)
aws cloudfront get-distribution-config --id $CFID > cfcfg.json
ETAG=$(jq -r .ETag cfcfg.json); jq '.DistributionConfig.Enabled=false | .DistributionConfig' cfcfg.json > cfdis.json
aws cloudfront update-distribution --id $CFID --if-match $ETAG --distribution-config file://cfdis.json --query Distribution.Status --output text
# 2. Web tier
aws autoscaling delete-auto-scaling-group --auto-scaling-group-name wordpress-asg --force-delete
ALB=$(aws elbv2 describe-load-balancers --names wordpress-alb --query "LoadBalancers[0].LoadBalancerArn" --output text)
aws elbv2 delete-load-balancer --load-balancer-arn $ALB
aws ec2 delete-launch-template --launch-template-name wordpress-lt --query LaunchTemplate.LaunchTemplateName --output text
AMI=$(aws ec2 describe-images --owners self --filters Name=name,Values=wordpress-ami --query "Images[0].ImageId" --output text)
SNAP=$(aws ec2 describe-images --image-ids $AMI --query "Images[0].BlockDeviceMappings[0].Ebs.SnapshotId" --output text)
aws ec2 deregister-image --image-id $AMI && aws ec2 delete-snapshot --snapshot-id $SNAP && echo AMI-DELETED
# 3. Databases (the RDS-managed secret is deleted together with wordpress-db)
aws rds delete-db-instance --db-instance-identifier wordpress-db-restored --skip-final-snapshot --delete-automated-backups --query DBInstance.DBInstanceStatus --output text
aws rds delete-db-instance --db-instance-identifier wordpress-db --skip-final-snapshot --delete-automated-backups --query DBInstance.DBInstanceStatus --output text
aws rds delete-db-snapshot --db-snapshot-identifier wordpress-db-snapshot --query DBSnapshot.Status --output text
```

#### Step 2 – Delete the network, the IAM role and the bucket

This step waits until the load balancer and both databases are deleted (about 10 minutes), then removes the VPC endpoints, security groups and the VPC.

```bash
VPC=$(aws ec2 describe-vpcs --filters Name=tag:Name,Values=wordpress-vpc --query "Vpcs[0].VpcId" --output text)
# 4. Wait until the load balancer and the databases are gone
aws elbv2 wait load-balancers-deleted --names wordpress-alb && sleep 30
TG=$(aws elbv2 describe-target-groups --names wordpress-tg --query "TargetGroups[0].TargetGroupArn" --output text)
aws elbv2 delete-target-group --target-group-arn $TG && echo TG-DELETED
aws rds wait db-instance-deleted --db-instance-identifier wordpress-db-restored
aws rds wait db-instance-deleted --db-instance-identifier wordpress-db
aws rds delete-db-subnet-group --db-subnet-group-name wordpress-db-subnet-group && echo SUBNET-GROUP-DELETED
# 5. VPC endpoints and security groups
EPS=$(aws ec2 describe-vpc-endpoints --filters Name=vpc-id,Values=$VPC --query "VpcEndpoints[].VpcEndpointId" --output text)
aws ec2 delete-vpc-endpoints --vpc-endpoint-ids $EPS --query "Unsuccessful" --output text
sleep 90
for g in Database-SG WebServer-SG ALB-SG Endpoint-SG; do
  GID=$(aws ec2 describe-security-groups --filters Name=vpc-id,Values=$VPC Name=group-name,Values=$g --query "SecurityGroups[0].GroupId" --output text)
  aws ec2 revoke-security-group-ingress --group-id $GID --ip-permissions "$(aws ec2 describe-security-groups --group-ids $GID --query 'SecurityGroups[0].IpPermissions' --output json)" > /dev/null
done
for g in Database-SG WebServer-SG ALB-SG Endpoint-SG; do
  GID=$(aws ec2 describe-security-groups --filters Name=vpc-id,Values=$VPC Name=group-name,Values=$g --query "SecurityGroups[0].GroupId" --output text)
  aws ec2 delete-security-group --group-id $GID && echo "$g deleted"
done
# 6. Subnets, route tables, internet gateway and VPC
for s in $(aws ec2 describe-subnets --filters Name=vpc-id,Values=$VPC --query "Subnets[].SubnetId" --output text); do aws ec2 delete-subnet --subnet-id $s; done
for r in $(aws ec2 describe-route-tables --filters Name=vpc-id,Values=$VPC --query "RouteTables[?Associations[0].Main!=\`true\`].RouteTableId" --output text); do aws ec2 delete-route-table --route-table-id $r; done
IGW=$(aws ec2 describe-internet-gateways --filters Name=attachment.vpc-id,Values=$VPC --query "InternetGateways[0].InternetGatewayId" --output text)
aws ec2 detach-internet-gateway --internet-gateway-id $IGW --vpc-id $VPC && aws ec2 delete-internet-gateway --internet-gateway-id $IGW
aws ec2 delete-vpc --vpc-id $VPC && echo VPC-DELETED
# 7. IAM role and S3 bucket
R=FCJ-WordPress-EC2-Role
aws iam remove-role-from-instance-profile --instance-profile-name $R --role-name $R
aws iam delete-instance-profile --instance-profile-name $R
aws iam detach-role-policy --role-name $R --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
aws iam delete-role-policy --role-name $R --policy-name WordPressAppAccess
aws iam delete-role --role-name $R && echo ROLE-DELETED
aws s3 rb s3://fcj-wordpress-files-1795416336 --force
```

{{% notice note %}}
If `delete-subnet` or `delete-vpc` reports `DependencyViolation`, the network interfaces of the VPC endpoints or the load balancer are still being removed. Wait one or two minutes and run the commands under `# 6.` again.
{{% /notice %}}

#### Step 3 – Delete the CloudFront distribution

A distribution can be deleted only after it has been disabled and the change is deployed (about 5–15 minutes).

```bash
# 8. CloudFront: delete the distribution after it shows Deployed with Enabled = false
CFID=$(aws cloudfront list-distributions --query "DistributionList.Items[?Comment=='WordPress on AWS (FCJ)'].Id" --output text)
aws cloudfront wait distribution-deployed --id $CFID
ETAG=$(aws cloudfront get-distribution --id $CFID --query ETag --output text)
aws cloudfront delete-distribution --id $CFID --if-match $ETAG && echo CF-DELETED
```

#### Check

- **EC2**: no instances, AMIs, snapshots, launch templates, load balancers or target groups named `wordpress-*`.
- **RDS**: no databases, snapshots or subnet groups named `wordpress-*`.
- **VPC**: `wordpress-vpc` and its endpoints are gone.
- **IAM**: the role `FCJ-WordPress-EC2-Role` is gone.
- **S3** and **CloudFront**: the bucket and the distribution are gone.
