---
title : "Launch the EC2 instance"
weight : 6
chapter : false
pre : " <b> 2.6 </b> "
---

You launch one instance, `wordpress-setup`, in a private subnet. You install WordPress on it in part 3 and then use it to create the AMI for the Auto Scaling group.

The instance uses:
- Amazon Linux 2023 for Arm (`t4g.micro`).
- `WebServer-SG` and the role `FCJ-WordPress-EC2-Role`.
- **No public IP address** (`--no-associate-public-ip-address`) and IMDSv2 only.

1. In **AWS CloudShell**, run:

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

   The output shows the instance ID, the private IP address and `None` for the public IP address.

2. Open the [Amazon EC2 console](https://console.aws.amazon.com/ec2/) → **Instances** → `wordpress-setup`. Check that:
   - (1) **Public IPv4 address** is empty.
   - (2) The instance is in a **private** subnet.
   - (3) **IMDSv2** is **Required**.

   ![Private EC2 instance](/images/2-prerequisite/2.6-ec2-private.png)
