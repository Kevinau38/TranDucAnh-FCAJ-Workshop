---
title : "Dọn dẹp tài nguyên"
weight : 8
chapter : false
pre : " <b> 8. </b> "
---

Đóng hai phiên Session Manager bằng **Terminate session**, rồi chạy các lệnh sau trong CloudShell theo đúng thứ tự.

1. Terminate các instance, xoá volume và xoá tám VPC endpoint:

   ```bash
   IDS=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=prod-postgres,test-postgres" "Name=instance-state-name,Values=pending,running,stopping,stopped" --query "Reservations[].Instances[].InstanceId" --output text)
   aws ec2 terminate-instances --instance-ids $IDS --query "TerminatingInstances[].[InstanceId,CurrentState.Name]" --output text
   aws ec2 wait instance-terminated --instance-ids $IDS && echo TERMINATED
   VOL=$(aws ec2 describe-volumes --filters Name=tag:Name,Values=shared-io2 --query "Volumes[0].VolumeId" --output text)
   aws ec2 delete-volume --volume-id $VOL && echo "DELETED $VOL"
   aws ec2 delete-vpc-endpoints --vpc-endpoint-ids $(aws ec2 describe-vpc-endpoints --filters "Name=tag:Name,Values=prod-*,test-*" --query "VpcEndpoints[].VpcEndpointId" --output text) --query "length(Unsuccessful)"
   ```

   Volume được gắn sau khi launch nên không bị xoá cùng instance. `delete-volume` chạy được khi cả hai instance đã terminate xong, và lệnh cuối in ra `0` khi mọi yêu cầu xoá endpoint đều được chấp nhận.

2. Interface endpoint cần vài phút để gỡ network interface. Chạy lệnh sau cho tới khi không còn in ra gì:

   ```bash
   aws ec2 describe-vpc-endpoints --filters "Name=tag:Name,Values=prod-*,test-*" --query "VpcEndpoints[?State!='deleted'].[Tags[0].Value,State]" --output text
   ```

3. Xoá security group, subnet, route table và hai VPC:

   ```bash
   for E in prod test; do
     VPC=$(aws ec2 describe-vpcs --filters Name=tag:Name,Values=$E-vpc --query "Vpcs[0].VpcId" --output text)
     for SG in $(aws ec2 describe-security-groups --filters Name=vpc-id,Values=$VPC --query "SecurityGroups[?GroupName!='default'].GroupId" --output text); do aws ec2 delete-security-group --group-id $SG > /dev/null; done
     for SN in $(aws ec2 describe-subnets --filters Name=vpc-id,Values=$VPC --query "Subnets[].SubnetId" --output text); do aws ec2 delete-subnet --subnet-id $SN; done
     for RT in $(aws ec2 describe-route-tables --filters Name=vpc-id,Values=$VPC --query "RouteTables[?Associations[0].Main!=\`true\`].RouteTableId" --output text); do aws ec2 delete-route-table --route-table-id $RT; done
     aws ec2 delete-vpc --vpc-id $VPC && echo "DELETED $E-vpc $VPC"
   done
   ```

   Vòng lặp in ra `DELETED prod-vpc ...` và `DELETED test-vpc ...`. Nếu gặp lỗi `DependencyViolation`, đợi thêm một phút rồi chạy lại vòng lặp.

4. Xoá instance profile và role:

   ```bash
   aws iam remove-role-from-instance-profile --instance-profile-name FCJ-NVMe-EC2-Role --role-name FCJ-NVMe-EC2-Role
   aws iam delete-instance-profile --instance-profile-name FCJ-NVMe-EC2-Role
   aws iam detach-role-policy --role-name FCJ-NVMe-EC2-Role --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
   aws iam delete-role --role-name FCJ-NVMe-EC2-Role && echo IAM-DELETED
   ```

5. Trong CloudShell, bạn có thể xoá file `ec2-trust.json`.
