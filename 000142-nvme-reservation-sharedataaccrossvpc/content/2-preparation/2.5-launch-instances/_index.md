---
title : "Launch the instances"
weight : 5
chapter : false
pre : " <b> 2.5 </b> "
---

You launch `prod-postgres` from the console, then `test-postgres` with the same settings from CloudShell.

1. Open the [Amazon EC2 console](https://console.aws.amazon.com/ec2/) and choose **Launch instance**.
   - (1) **Name**: `prod-postgres`.
   - (2) **Application and OS Images**: **Amazon Linux**.
   - (3) Keep the default **Amazon Linux 2023** AMI, 64-bit (x86).

   ![Name and AMI](/images/2-preparation/2.5-name-ami.png)

2. Scroll down:
   - (1) **Instance type**: `t3.micro`. T3 instances are built on the Nitro System, which Multi-Attach requires.
   - (2) **Key pair**: **Proceed without a key pair**. You connect with Session Manager, so you do not need SSH.

   ![Instance type and key pair](/images/2-preparation/2.5-type-keypair.png)

3. In **Network settings**, choose **Edit**:
   - (1) **VPC**: `prod-vpc`.
   - (2) **Subnet**: `prod-subnet-private1-ap-northeast-1a`.
   - (3) **Auto-assign public IP**: **Disable**.
   - (4) **Firewall**: **Select existing security group** → `prod-instance-sg`.

   ![Network settings](/images/2-preparation/2.5-network.png)

4. Open **Advanced details**:
   - (1) **IAM instance profile**: `FCJ-NVMe-EC2-Role`.
   - (2) Choose **Launch instance**.

   ![IAM instance profile](/images/2-preparation/2.5-iam-profile.png)

5. In CloudShell, launch `test-postgres` in `test-vpc` with the same AMI, instance type and role:

   ```bash
   VPC=$(aws ec2 describe-vpcs --filters Name=tag:Name,Values=test-vpc --query "Vpcs[0].VpcId" --output text)
   SUBNET=$(aws ec2 describe-subnets --filters Name=vpc-id,Values=$VPC --query "Subnets[0].SubnetId" --output text)
   SG=$(aws ec2 describe-security-groups --filters Name=vpc-id,Values=$VPC Name=group-name,Values=test-instance-sg --query "SecurityGroups[0].GroupId" --output text)
   AMI=$(aws ssm get-parameter --name /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64 --query Parameter.Value --output text)
   aws ec2 run-instances --image-id $AMI --instance-type t3.micro --subnet-id $SUBNET --security-group-ids $SG --iam-instance-profile Name=FCJ-NVMe-EC2-Role --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=test-postgres}]" --query "Instances[0].[InstanceId,Placement.AvailabilityZone,PrivateIpAddress]" --output text
   ```

6. Open **Instances**:
   - (1) `prod-postgres` and `test-postgres` are **Running**.
   - (2) Both are in `ap-northeast-1a`.
   - (3) Neither has a public IPv4 address.

   ![Instances](/images/2-preparation/2.5-instances.png)

Wait until **Status check** shows **3/3 checks passed** before you continue.
