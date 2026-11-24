---
title : "AWS Backup"
weight : 6
chapter : false
pre : " <b> 4.6. </b> "
---

#### Tổng quan

**AWS Backup** quản lý việc sao lưu của nhiều dịch vụ AWS (RDS, Aurora, DynamoDB, EBS, EFS, S3...) tại một nơi. Thay vì cấu hình từng cơ sở dữ liệu, bạn viết một **backup plan** một lần và gán tài nguyên cho nó, ví dụ mọi cơ sở dữ liệu RDS có một tag nhất định.

| Thành phần | Tên trong lab | Mục đích |
|---|---|---|
| IAM role | `FCJ-rdspg-backup-role` | Cho phép AWS Backup tạo và khôi phục recovery point |
| Backup vault | `rdspg-backup-vault` | Lưu trữ và mã hóa các recovery point |
| On-demand backup | — | Một lần sao lưu, chạy ngay |
| Backup plan | `rdspg-backup-plan`, rule `rdspg-daily` | Sao lưu hằng ngày lúc 00:30, giữ 7 ngày |
| Resource assignment | `rdspg-tagged-resources` | Mọi cơ sở dữ liệu RDS có tag `backup-plan` = `rdspg-daily` |

#### Tạo IAM role

AWS Backup có thể tự tạo role mặc định. Trong lab này bạn tự tạo role để thấy rõ AWS Backup được cấp những quyền nào.

1. Mở [IAM console](https://console.aws.amazon.com/iam/home#/roles), chọn **Roles** > **Create role**:
    - (1) **Trusted entity type**: **AWS service**
    - (2) **Service or use case**: **AWS Backup**
    - (3) **Use case**: **AWS Backup**
    - (4) Chọn **Next**.

    ![Trusted entity](/images/4/4-6/1.png)

2. Nhập `AWSBackupServiceRolePolicyFor` vào ô tìm kiếm:
    - (1) Danh sách hiện các AWS managed policy dành cho AWS Backup.
    - (2) Chọn `AWSBackupServiceRolePolicyForBackup`.
    - (3) Chọn `AWSBackupServiceRolePolicyForRestores`.

    Chọn **Next**.

    ![Add permissions](/images/4/4-6/2.png)

3. Trên trang **Name, review, and create**:
    - (1) **Role name**: `FCJ-rdspg-backup-role`

    ![Role name](/images/4/4-6/3.png)

4. Kéo xuống:
    - (1) Kiểm tra hai policy được gắn dưới dạng **Permissions policy**.
    - (2) Chọn **Create role**.

    ![Create role](/images/4/4-6/4.png)

{{% notice note %}}
Nếu tài khoản AWS của bạn yêu cầu **permissions boundary** cho role mới (như tài khoản dùng chung khi viết lab này), hãy đặt nó trong **Set permissions boundary** ở trang **Add permissions**. Khi đó nó hiện là **Boundary policy** trong phần tóm tắt.
{{% /notice %}}

#### Tạo backup vault

5. Mở [AWS Backup console](https://console.aws.amazon.com/backup/home#/backupvaults) ở Region của cơ sở dữ liệu. Chọn **Vaults** > **Create new vault**:
    - (1) **Vault name**: `rdspg-backup-vault`
    - (2) **Vault type**: **Backup vault**
    - (3) **Encryption key**: `(default) aws/backup`

    Chọn **Create vault** ở cuối trang.

    ![Create vault](/images/4/4-6/5.png)

#### Chạy on-demand backup

6. Chọn **Protected resources** > **Create on-demand backup**:
    - (1) **Resource type**: **RDS**
    - (2) **Database name**: `rdspg-lab-db`
    - (3) **Backup window**: **Create backup now**
    - (4) **Total retention period**: `1` Days

    ![On-demand backup settings](/images/4/4-6/6.png)

7. Kéo xuống:
    - (1) **Backup vault**: `rdspg-backup-vault`
    - (2) **IAM role**: **Use another role**
    - (3) Chọn `FCJ-rdspg-backup-role`.
    - (4) Chọn **Create on-demand backup**.

    ![On-demand backup vault and role](/images/4/4-6/7.png)

{{% notice info %}}
Console cảnh báo thời gian lưu dưới 7 ngày có thể phát sinh thêm phí. Cảnh báo này áp dụng cho các loại tài nguyên có thời gian lưu tối thiểu. Với lab này, 1 ngày là đủ, và bạn xóa recovery point ở bước Dọn dẹp.
{{% /notice %}}

8. AWS Backup mở trang **Jobs**. Sau khoảng 10 phút, job ở trạng thái **Completed**. Chọn **Vaults** > `rdspg-backup-vault`:
    - (1) Recovery point của `rdspg-lab-db` ở trạng thái **Completed**, loại sao lưu **Snapshot**.

    ![Recovery point](/images/4/4-6/8.png)

#### Tạo backup plan

9. Chọn **Backup plans** > **Create backup plan**:
    - (1) **Start options**: **Build a new plan**
    - (2) **Backup plan name**: `rdspg-backup-plan`
    - (3) **Backup rule name**: `rdspg-daily`

    ![Backup plan](/images/4/4-6/9.png)

10. Tiếp tục trong backup rule:
    - (1) **Backup vault**: `rdspg-backup-vault`
    - (2) **Backup frequency**: **Daily**
    - (3) **Start time**: giữ mặc định `00:30` theo múi giờ của bạn.

    ![Backup rule schedule](/images/4/4-6/10.png)

11. Kéo xuống:
    - (1) Không chọn **Enable continuous backups for point-in-time recovery (PITR)**. Automated backup của RDS đã cho bạn khả năng khôi phục về một thời điểm.
    - (2) **Total retention period**: `7` Days

    Chọn **Create plan** ở cuối trang.

    ![Backup rule retention](/images/4/4-6/11.png)

#### Gán tài nguyên theo tag

12. AWS Backup mở trang **Assign resources**:
    - (1) **Resource assignment name**: `rdspg-tagged-resources`
    - (2) **IAM role**: **Use another role**
    - (3) Chọn `FCJ-rdspg-backup-role`.

    ![Assign resources](/images/4/4-6/12.png)

13. Kéo xuống:
    - (1) **Define resource selection**: **Include specific resource types**
    - (2) **Resource type**: **RDS**, **Database names**: **All databases**
    - (3) **Refine selection using tags**: **Key** `backup-plan`, **Condition for value** **Equals**, **Value** `rdspg-daily`

    Chọn **Assign resources**, rồi **Continue** trong hộp xác nhận.

    ![Resource selection](/images/4/4-6/13.png)

{{% notice tip %}}
Điều kiện tag rất quan trọng. Nếu thiếu, plan sẽ sao lưu **mọi** cơ sở dữ liệu RDS trong tài khoản và Region.
{{% /notice %}}

14. Backup plan đã sẵn sàng:
    - (1) Rule `rdspg-daily` lưu bản sao lưu vào `rdspg-backup-vault`.
    - (2) Resource assignment `rdspg-tagged-resources` dùng `FCJ-rdspg-backup-role`.

    ![Backup plan details](/images/4/4-6/14.png)

15. Gắn tag cho cơ sở dữ liệu. Trong RDS console, chọn `rdspg-lab-db` > **Tags** > **Manage tags** > **Add new tag**:
    - (1) **Key**: `backup-plan`
    - (2) **Value**: `rdspg-daily`
    - (3) Chọn **Save changes**.

    ![Tag the database](/images/4/4-6/15.png)

Từ giờ, AWS Backup sao lưu `rdspg-lab-db` mỗi ngày lúc 00:30. Để thêm một cơ sở dữ liệu khác vào plan, bạn chỉ cần gắn cùng tag.

#### (Tùy chọn) AWS CLI

{{%expand "Chạy on-demand backup bằng AWS CLI" %}}
```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
DB_ARN=$(aws rds describe-db-instances --db-instance-identifier rdspg-lab-db \
    --query 'DBInstances[0].DBInstanceArn' --output text)

aws backup create-backup-vault --backup-vault-name rdspg-backup-vault

aws backup start-backup-job \
    --backup-vault-name rdspg-backup-vault \
    --resource-arn $DB_ARN \
    --iam-role-arn arn:aws:iam::$ACCOUNT_ID:role/FCJ-rdspg-backup-role \
    --lifecycle DeleteAfterDays=1

aws backup list-backup-jobs --by-backup-vault-name rdspg-backup-vault \
    --query 'BackupJobs[].[State,PercentDone,CreationDate]' --output table
```
{{% /expand %}}
