---
title : "Tạo cơ sở dữ liệu RDS"
weight : 7
chapter : false
pre : " <b> 2.7 </b> "
---

Bạn tạo một cơ sở dữ liệu Amazon RDS for MySQL 8.4 trong các private subnet:

| Cấu hình | Giá trị | Lý do |
|---|---|---|
| DB subnet group | `wordpress-db-subnet-group` (hai private subnet) | Cơ sở dữ liệu không bao giờ nằm trong public subnet |
| Multi-AZ | Có | RDS giữ một bản standby đồng bộ ở Availability Zone còn lại |
| Public access | Không | Chỉ `WebServer-SG` kết nối được, thông qua `Database-SG` |
| Master password | `--manage-master-user-password` | RDS tự tạo mật khẩu và lưu trong AWS Secrets Manager, không ai phải gõ hay sao chép mật khẩu |
| Database ban đầu | `wordpress` | WordPress dùng database này |
| Enhanced Monitoring | Tắt (`--monitoring-interval 0`) | Enhanced Monitoring cần thêm một IAM role |

1. Trong **AWS CloudShell**, chạy:

   ```bash
   VPC=$(aws ec2 describe-vpcs --filters Name=tag:Name,Values=wordpress-vpc --query "Vpcs[0].VpcId" --output text)
   SUBNETS=$(aws ec2 describe-subnets --filters Name=vpc-id,Values=$VPC Name=tag:Name,Values="*private*" --query "Subnets[].SubnetId" --output text)
   DBSG=$(aws ec2 describe-security-groups --filters Name=vpc-id,Values=$VPC Name=group-name,Values=Database-SG --query "SecurityGroups[0].GroupId" --output text)
   aws rds create-db-subnet-group --db-subnet-group-name wordpress-db-subnet-group \
     --db-subnet-group-description "Private subnets for the WordPress database" \
     --subnet-ids $SUBNETS --query DBSubnetGroup.SubnetGroupStatus --output text
   aws rds create-db-instance --db-instance-identifier wordpress-db \
     --engine mysql --engine-version 8.4.11 --db-instance-class db.t4g.micro \
     --allocated-storage 20 --storage-type gp3 --storage-encrypted \
     --master-username admin --manage-master-user-password \
     --multi-az --db-subnet-group-name wordpress-db-subnet-group \
     --vpc-security-group-ids $DBSG --db-name wordpress \
     --backup-retention-period 1 --no-publicly-accessible \
     --monitoring-interval 0 --no-deletion-protection \
     --query "DBInstance.[DBInstanceIdentifier,DBInstanceStatus,MultiAZ]" --output text
   ```

2. Mở [Amazon RDS console](https://console.aws.amazon.com/rds/) → **Databases** → `wordpress-db`. Việc tạo instance Multi-AZ mất khoảng 15 phút. Bạn có thể làm tiếp phần 3 khi trạng thái đang là **Creating**; bước cấu hình WordPress chỉ cần cơ sở dữ liệu đã có endpoint.

3. Khi trạng thái là **Available**, mở tab **Configuration** để kiểm tra (1) phiên bản engine `8.4.x`, (2) tên database `wordpress`, (3) **Multi-AZ** = **Yes** và (4) **Enhanced Monitoring** = **Disabled**.

   ![Cơ sở dữ liệu RDS](/images/2-prerequisite/2.7-rds.png)
