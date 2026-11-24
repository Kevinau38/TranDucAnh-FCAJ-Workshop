---
title : "Dọn dẹp tài nguyên"
weight : 7
chapter : false
pre : " <b> 7. </b> "
---

Xoá tài nguyên theo thứ tự dưới đây. Một số tài nguyên phải được xoá hẳn thì mới xoá được các tài nguyên tiếp theo, vì vậy các câu lệnh được chia thành ba bước. Chạy tất cả trong **AWS CloudShell**.

#### Bước 1 – Tắt CloudFront, xoá tầng web và cơ sở dữ liệu

```bash
# 1. CloudFront: tắt distribution
CFID=$(aws cloudfront list-distributions --query "DistributionList.Items[?Comment=='WordPress on AWS (FCJ)'].Id" --output text)
aws cloudfront get-distribution-config --id $CFID > cfcfg.json
ETAG=$(jq -r .ETag cfcfg.json); jq '.DistributionConfig.Enabled=false | .DistributionConfig' cfcfg.json > cfdis.json
aws cloudfront update-distribution --id $CFID --if-match $ETAG --distribution-config file://cfdis.json --query Distribution.Status --output text
# 2. Tầng web
aws autoscaling delete-auto-scaling-group --auto-scaling-group-name wordpress-asg --force-delete
ALB=$(aws elbv2 describe-load-balancers --names wordpress-alb --query "LoadBalancers[0].LoadBalancerArn" --output text)
aws elbv2 delete-load-balancer --load-balancer-arn $ALB
aws ec2 delete-launch-template --launch-template-name wordpress-lt --query LaunchTemplate.LaunchTemplateName --output text
AMI=$(aws ec2 describe-images --owners self --filters Name=name,Values=wordpress-ami --query "Images[0].ImageId" --output text)
SNAP=$(aws ec2 describe-images --image-ids $AMI --query "Images[0].BlockDeviceMappings[0].Ebs.SnapshotId" --output text)
aws ec2 deregister-image --image-id $AMI && aws ec2 delete-snapshot --snapshot-id $SNAP && echo AMI-DELETED
# 3. Cơ sở dữ liệu (secret do RDS quản lý được xoá cùng wordpress-db)
aws rds delete-db-instance --db-instance-identifier wordpress-db-restored --skip-final-snapshot --delete-automated-backups --query DBInstance.DBInstanceStatus --output text
aws rds delete-db-instance --db-instance-identifier wordpress-db --skip-final-snapshot --delete-automated-backups --query DBInstance.DBInstanceStatus --output text
aws rds delete-db-snapshot --db-snapshot-identifier wordpress-db-snapshot --query DBSnapshot.Status --output text
```

#### Bước 2 – Xoá mạng, IAM role và bucket

Bước này chờ đến khi load balancer và cả hai cơ sở dữ liệu đã bị xoá (khoảng 10 phút), sau đó xoá các VPC endpoint, security group và VPC.

```bash
VPC=$(aws ec2 describe-vpcs --filters Name=tag:Name,Values=wordpress-vpc --query "Vpcs[0].VpcId" --output text)
# 4. Chờ load balancer và cơ sở dữ liệu bị xoá hẳn
aws elbv2 wait load-balancers-deleted --names wordpress-alb && sleep 30
TG=$(aws elbv2 describe-target-groups --names wordpress-tg --query "TargetGroups[0].TargetGroupArn" --output text)
aws elbv2 delete-target-group --target-group-arn $TG && echo TG-DELETED
aws rds wait db-instance-deleted --db-instance-identifier wordpress-db-restored
aws rds wait db-instance-deleted --db-instance-identifier wordpress-db
aws rds delete-db-subnet-group --db-subnet-group-name wordpress-db-subnet-group && echo SUBNET-GROUP-DELETED
# 5. VPC endpoint và security group
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
# 6. Subnet, route table, internet gateway và VPC
for s in $(aws ec2 describe-subnets --filters Name=vpc-id,Values=$VPC --query "Subnets[].SubnetId" --output text); do aws ec2 delete-subnet --subnet-id $s; done
for r in $(aws ec2 describe-route-tables --filters Name=vpc-id,Values=$VPC --query "RouteTables[?Associations[0].Main!=\`true\`].RouteTableId" --output text); do aws ec2 delete-route-table --route-table-id $r; done
IGW=$(aws ec2 describe-internet-gateways --filters Name=attachment.vpc-id,Values=$VPC --query "InternetGateways[0].InternetGatewayId" --output text)
aws ec2 detach-internet-gateway --internet-gateway-id $IGW --vpc-id $VPC && aws ec2 delete-internet-gateway --internet-gateway-id $IGW
aws ec2 delete-vpc --vpc-id $VPC && echo VPC-DELETED
# 7. IAM role và bucket S3
R=FCJ-WordPress-EC2-Role
aws iam remove-role-from-instance-profile --instance-profile-name $R --role-name $R
aws iam delete-instance-profile --instance-profile-name $R
aws iam detach-role-policy --role-name $R --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
aws iam delete-role-policy --role-name $R --policy-name WordPressAppAccess
aws iam delete-role --role-name $R && echo ROLE-DELETED
aws s3 rb s3://fcj-wordpress-files-1795416336 --force
```

{{% notice note %}}
Nếu `delete-subnet` hoặc `delete-vpc` báo lỗi `DependencyViolation`, các network interface của VPC endpoint hoặc load balancer vẫn đang được xoá. Hãy chờ một đến hai phút rồi chạy lại các câu lệnh ở mục `# 6.`.
{{% /notice %}}

#### Bước 3 – Xoá CloudFront distribution

Distribution chỉ xoá được sau khi đã bị tắt (disable) và thay đổi đã được triển khai xong (khoảng 5–15 phút).

```bash
# 8. CloudFront: xoá distribution sau khi trạng thái là Deployed và Enabled = false
CFID=$(aws cloudfront list-distributions --query "DistributionList.Items[?Comment=='WordPress on AWS (FCJ)'].Id" --output text)
aws cloudfront wait distribution-deployed --id $CFID
ETAG=$(aws cloudfront get-distribution --id $CFID --query ETag --output text)
aws cloudfront delete-distribution --id $CFID --if-match $ETAG && echo CF-DELETED
```

#### Kiểm tra

- **EC2**: không còn instance, AMI, snapshot, launch template, load balancer hay target group nào tên `wordpress-*`.
- **RDS**: không còn database, snapshot hay subnet group nào tên `wordpress-*`.
- **VPC**: `wordpress-vpc` và các endpoint đã bị xoá.
- **IAM**: role `FCJ-WordPress-EC2-Role` đã bị xoá.
- **S3** và **CloudFront**: bucket và distribution đã bị xoá.
