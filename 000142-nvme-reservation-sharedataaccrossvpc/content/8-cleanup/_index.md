---
title : "Clean up resources"
weight : 8
chapter : false
pre : " <b> 8. </b> "
---

Close the two Session Manager sessions with **Terminate session**, then run the following commands in CloudShell, in this order.

1. Terminate the instances, delete the volume, and delete the eight VPC endpoints:

   ```bash
   IDS=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=prod-postgres,test-postgres" "Name=instance-state-name,Values=pending,running,stopping,stopped" --query "Reservations[].Instances[].InstanceId" --output text)
   aws ec2 terminate-instances --instance-ids $IDS --query "TerminatingInstances[].[InstanceId,CurrentState.Name]" --output text
   aws ec2 wait instance-terminated --instance-ids $IDS && echo TERMINATED
   VOL=$(aws ec2 describe-volumes --filters Name=tag:Name,Values=shared-io2 --query "Volumes[0].VolumeId" --output text)
   aws ec2 delete-volume --volume-id $VOL && echo "DELETED $VOL"
   aws ec2 delete-vpc-endpoints --vpc-endpoint-ids $(aws ec2 describe-vpc-endpoints --filters "Name=tag:Name,Values=prod-*,test-*" --query "VpcEndpoints[].VpcEndpointId" --output text) --query "length(Unsuccessful)"
   ```

   The volume was attached after launch, so it is not deleted with the instances. `delete-volume` works once both instances are terminated, and the last command prints `0` when every endpoint deletion is accepted.

2. Interface endpoints take a few minutes to remove their network interfaces. Run this command until it prints nothing:

   ```bash
   aws ec2 describe-vpc-endpoints --filters "Name=tag:Name,Values=prod-*,test-*" --query "VpcEndpoints[?State!='deleted'].[Tags[0].Value,State]" --output text
   ```

3. Delete the security groups, subnets, route tables and the two VPCs:

   ```bash
   for E in prod test; do
     VPC=$(aws ec2 describe-vpcs --filters Name=tag:Name,Values=$E-vpc --query "Vpcs[0].VpcId" --output text)
     for SG in $(aws ec2 describe-security-groups --filters Name=vpc-id,Values=$VPC --query "SecurityGroups[?GroupName!='default'].GroupId" --output text); do aws ec2 delete-security-group --group-id $SG > /dev/null; done
     for SN in $(aws ec2 describe-subnets --filters Name=vpc-id,Values=$VPC --query "Subnets[].SubnetId" --output text); do aws ec2 delete-subnet --subnet-id $SN; done
     for RT in $(aws ec2 describe-route-tables --filters Name=vpc-id,Values=$VPC --query "RouteTables[?Associations[0].Main!=\`true\`].RouteTableId" --output text); do aws ec2 delete-route-table --route-table-id $RT; done
     aws ec2 delete-vpc --vpc-id $VPC && echo "DELETED $E-vpc $VPC"
   done
   ```

   The loop prints `DELETED prod-vpc ...` and `DELETED test-vpc ...`. If you see `DependencyViolation`, wait one more minute and run the loop again.

4. Delete the instance profile and the role:

   ```bash
   aws iam remove-role-from-instance-profile --instance-profile-name FCJ-NVMe-EC2-Role --role-name FCJ-NVMe-EC2-Role
   aws iam delete-instance-profile --instance-profile-name FCJ-NVMe-EC2-Role
   aws iam detach-role-policy --role-name FCJ-NVMe-EC2-Role --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
   aws iam delete-role --role-name FCJ-NVMe-EC2-Role && echo IAM-DELETED
   ```

5. In CloudShell, you can delete the file `ec2-trust.json`.
