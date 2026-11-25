---
title : "Tạo các instance"
weight : 5
chapter : false
pre : " <b> 2.5 </b> "
---

Bạn tạo `prod-postgres` trên console, sau đó tạo `test-postgres` với cùng cấu hình bằng CloudShell.

1. Mở [Amazon EC2 console](https://console.aws.amazon.com/ec2/) và chọn **Launch instance**.
   - (1) **Name**: `prod-postgres`.
   - (2) **Application and OS Images**: **Amazon Linux**.
   - (3) Giữ AMI mặc định **Amazon Linux 2023**, 64-bit (x86).

   ![Tên và AMI](/images/2-preparation/2.5-name-ami.png)

2. Cuộn xuống:
   - (1) **Instance type**: `t3.micro`. Dòng T3 chạy trên Nitro System, là điều kiện bắt buộc của Multi-Attach.
   - (2) **Key pair**: **Proceed without a key pair**. Bạn kết nối bằng Session Manager nên không cần SSH.

   ![Instance type và key pair](/images/2-preparation/2.5-type-keypair.png)

3. Ở **Network settings**, chọn **Edit**:
   - (1) **VPC**: `prod-vpc`.
   - (2) **Subnet**: `prod-subnet-private1-ap-northeast-1a`.
   - (3) **Auto-assign public IP**: **Disable**.
   - (4) **Firewall**: **Select existing security group** → `prod-instance-sg`.

   ![Network settings](/images/2-preparation/2.5-network.png)

4. Mở **Advanced details**:
   - (1) **IAM instance profile**: `FCJ-NVMe-EC2-Role`.
   - (2) Chọn **Launch instance**.

   ![IAM instance profile](/images/2-preparation/2.5-iam-profile.png)

5. Trong CloudShell, tạo `test-postgres` trong `test-vpc` với cùng AMI, instance type và role:

   ```bash
   VPC=$(aws ec2 describe-vpcs --filters Name=tag:Name,Values=test-vpc --query "Vpcs[0].VpcId" --output text)
   SUBNET=$(aws ec2 describe-subnets --filters Name=vpc-id,Values=$VPC --query "Subnets[0].SubnetId" --output text)
   SG=$(aws ec2 describe-security-groups --filters Name=vpc-id,Values=$VPC Name=group-name,Values=test-instance-sg --query "SecurityGroups[0].GroupId" --output text)
   AMI=$(aws ssm get-parameter --name /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64 --query Parameter.Value --output text)
   aws ec2 run-instances --image-id $AMI --instance-type t3.micro --subnet-id $SUBNET --security-group-ids $SG --iam-instance-profile Name=FCJ-NVMe-EC2-Role --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=test-postgres}]" --query "Instances[0].[InstanceId,Placement.AvailabilityZone,PrivateIpAddress]" --output text
   ```

6. Mở **Instances**:
   - (1) `prod-postgres` và `test-postgres` ở trạng thái **Running**.
   - (2) Cả hai nằm ở `ap-northeast-1a`.
   - (3) Không instance nào có public IPv4.

   ![Các instance](/images/2-preparation/2.5-instances.png)

Đợi cột **Status check** báo **3/3 checks passed** rồi mới làm tiếp.
