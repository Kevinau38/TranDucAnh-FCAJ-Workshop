---
title : "Sao lưu và khôi phục cơ sở dữ liệu"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

Multi-AZ bảo vệ cơ sở dữ liệu khi một Availability Zone gặp sự cố, nhưng không bảo vệ bạn khỏi một thay đổi sai: một bài viết bị xoá thì cũng bị xoá trên bản standby. **DB snapshot** giữ một bản sao tại một thời điểm mà bạn có thể khôi phục thành một DB instance mới.

#### Tạo snapshot

1. Trong **AWS CloudShell**, chạy:

   ```bash
   aws rds create-db-snapshot --db-instance-identifier wordpress-db --db-snapshot-identifier wordpress-db-snapshot --query "DBSnapshot.[DBSnapshotIdentifier,Status]" --output text
   ```

2. Mở [Amazon RDS console](https://console.aws.amazon.com/rds/) → **Snapshots** → **Manual**, rồi (1) lọc theo `wordpress-db`. Sau vài phút, snapshot ở trạng thái (2) **Available** và tiến độ là **Completed**.

   ![DB snapshot](/images/5-backup/5.1-snapshot.png)

#### Khôi phục snapshot thành instance mới

Khôi phục luôn tạo ra một DB instance **mới**. Bạn khôi phục vào cùng DB subnet group private và `Database-SG`. Để tiết kiệm chi phí, bản khôi phục chỉ dùng Single-AZ.

1. Trong **AWS CloudShell**, chạy:

   ```bash
   VPC=$(aws ec2 describe-vpcs --filters Name=tag:Name,Values=wordpress-vpc --query "Vpcs[0].VpcId" --output text)
   DBSG=$(aws ec2 describe-security-groups --filters Name=vpc-id,Values=$VPC Name=group-name,Values=Database-SG --query "SecurityGroups[0].GroupId" --output text)
   aws rds restore-db-instance-from-db-snapshot --db-instance-identifier wordpress-db-restored \
     --db-snapshot-identifier wordpress-db-snapshot --db-instance-class db.t4g.micro \
     --db-subnet-group-name wordpress-db-subnet-group --vpc-security-group-ids $DBSG \
     --no-multi-az --no-publicly-accessible --no-deletion-protection \
     --query "DBInstance.[DBInstanceIdentifier,DBInstanceStatus,MultiAZ]" --output text
   ```

2. Mở **Databases**. Sau khoảng 10 phút, (1) `wordpress-db-restored` ở trạng thái **Available** bên cạnh `wordpress-db` gốc, trong cùng DB subnet group private.

   ![Instance đã khôi phục](/images/5-backup/5.2-restored.png)

{{% notice tip %}}
Để chuyển WordPress sang cơ sở dữ liệu đã khôi phục, bạn đổi `DB_HOST` trong `wp-config.php` thành endpoint của `wordpress-db-restored`, tạo AMI mới và cập nhật launch template. Trong bài lab này bạn chỉ kiểm tra việc khôi phục, và sẽ xoá instance khôi phục ở bước dọn dẹp.
{{% /notice %}}
