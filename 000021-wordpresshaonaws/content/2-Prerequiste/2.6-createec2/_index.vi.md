---
title : "Khởi tạo EC2 instance"
weight : 6
chapter : false
pre : " <b> 2.6 </b> "
---

Bạn khởi tạo một instance tên `wordpress-setup` trong private subnet. Ở phần 3, bạn cài WordPress lên instance này rồi dùng nó để tạo AMI cho Auto Scaling group.

Instance sử dụng:
- Amazon Linux 2023 cho Arm (`t4g.micro`).
- `WebServer-SG` và role `FCJ-WordPress-EC2-Role`.
- **Không có địa chỉ IP public** (`--no-associate-public-ip-address`) và chỉ dùng IMDSv2.

1. Trong **AWS CloudShell**, chạy:

   ```bash
   VPC=$(aws ec2 describe-vpcs --filters Name=tag:Name,Values=wordpress-vpc --query "Vpcs[0].VpcId" --output text)
   SUB1=$(aws ec2 describe-subnets --filters Name=vpc-id,Values=$VPC Name=tag:Name,Values="*private2*" --query "Subnets[0].SubnetId" --output text)
   WEBSG=$(aws ec2 describe-security-groups --filters Name=vpc-id,Values=$VPC Name=group-name,Values=WebServer-SG --query "SecurityGroups[0].GroupId" --output text)
   AMI=$(aws ssm get-parameter --name /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-arm64 --query Parameter.Value --output text)
   aws ec2 run-instances --image-id $AMI --instance-type t4g.micro \
     --subnet-id $SUB1 --security-group-ids $WEBSG \
     --iam-instance-profile Name=FCJ-WordPress-EC2-Role \
     --no-associate-public-ip-address \
     --metadata-options HttpTokens=required,HttpEndpoint=enabled \
     --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=wordpress-setup}]" \
     --query "Instances[0].[InstanceId,PrivateIpAddress,PublicIpAddress]" --output text
   ```

   Kết quả hiển thị instance ID, địa chỉ IP private và `None` cho địa chỉ IP public.

2. Mở [Amazon EC2 console](https://console.aws.amazon.com/ec2/) → **Instances** → `wordpress-setup`. Kiểm tra:
   - (1) **Public IPv4 address** để trống.
   - (2) Instance nằm trong subnet **private**.
   - (3) **IMDSv2** là **Required**.

   ![EC2 instance private](/images/2-prerequisite/2.6-ec2-private.png)
