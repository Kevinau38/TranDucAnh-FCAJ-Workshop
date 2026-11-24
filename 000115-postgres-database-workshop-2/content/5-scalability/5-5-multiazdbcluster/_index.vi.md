---
title : "Tạo DB instance read replica từ Multi-AZ DB cluster"
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

#### Tổng quan

Hai standby đọc được của Multi-AZ DB cluster nằm trong cùng Region và dùng chung reader endpoint. Khi cần thêm năng lực đọc, hoặc một bản sao dữ liệu có thể mở rộng riêng, bạn có thể thêm một **DB instance read replica** cho cluster.

Ở bước này, bạn tạo read replica `rdspg-lab-cluster-read` từ `rdspg-lab-cluster`, kiểm tra replication, rồi xoá replica và cluster để dừng chi phí.

{{% notice note %}}
Để làm nguồn replication, Multi-AZ DB cluster phải ở trạng thái **Available** và bật automated backup. Ngay sau khi promote ở bước 5.4, RDS tạo bản backup đầu tiên (**Backing-up**) và thao tác **Create read replica** bị vô hiệu cho đến khi backup xong.
{{% /notice %}}

#### Tạo read replica

1. Trong danh sách database:
    - (1) Chọn `rdspg-lab-cluster`.
    - (2) Chọn **Actions**.
    - (3) Chọn **Create read replica**.

    ![Create read replica](/images/5/5-5/1.png)

2. Trên trang **Create read replica**:
    - (1) **Replica source** là `rdspg-lab-cluster`, role **Multi-AZ DB cluster**.
    - (2) **DB identifier**: `rdspg-lab-cluster-read`
    - (3) **DB instance class**: **Burstable classes**
    - (4) **Instance type**: `db.t4g.micro`. Read replica của Multi-AZ DB cluster không bắt buộc dùng cùng class với cluster.

    ![Cấu hình replica](/images/5/5-5/2.png)

3. Kéo xuống và kiểm tra các cấu hình còn lại:
    - **Availability**: **Single-AZ DB instance deployment (1 instance)**
    - **DB subnet group**: `rdspg-lab-subnet-group`
    - **Public access**: **Not publicly accessible**
    - **Existing VPC security groups**: `rds-sg`

    Chọn **Create read replica**.

4. Trong danh sách database:
    - (1) `rdspg-lab-cluster-read` xuất hiện dưới `rdspg-lab-cluster` với role **Replica** và trạng thái **Creating**.

    ![Đang tạo replica](/images/5/5-5/3.png)

5. Sau khoảng 15 phút, chọn `rdspg-lab-cluster-read` và mở tab **Replication**:
    - (1) Breadcrumb cho thấy replica thuộc về `rdspg-lab-cluster`.
    - (2) **Replication state** là **Replicating**.

    ![Replication](/images/5/5-5/4.png)

#### Xoá Multi-AZ DB cluster

Cluster và replica không thuộc free tier. Hãy xoá ngay. Bạn không cần final snapshot.

6. Mở [AWS CloudShell](https://console.aws.amazon.com/cloudshell/) và chạy:

    ```bash
    aws rds delete-db-instance --db-instance-identifier rdspg-lab-cluster-read \
        --skip-final-snapshot --delete-automated-backups \
        --query 'DBInstance.[DBInstanceIdentifier,DBInstanceStatus]' --output text

    aws rds delete-db-cluster --db-cluster-identifier rdspg-lab-cluster \
        --skip-final-snapshot --delete-automated-backups \
        --query 'DBCluster.[DBClusterIdentifier,Status]' --output text

    aws rds describe-db-clusters --db-cluster-identifier rdspg-lab-cluster \
        --query 'DBClusters[0].Status' --output text
    ```

    - (1) Replica `rdspg-lab-cluster-read` đang **deleting**.
    - (2) RDS chấp nhận yêu cầu xoá cluster. Lệnh vẫn hiển thị trạng thái trước khi xoá, **available**.
    - (3) Vài giây sau, trạng thái của cluster là **deleting**. RDS xoá luôn ba DB instance của nó. Sau khoảng 10 phút, `describe-db-clusters` trả về *DBClusterNotFoundFault*.

    ![Xoá cluster](/images/5/5-5/5.png)

{{% notice tip %}}
Trong console, bạn xoá Multi-AZ DB cluster bằng cách chọn cluster (không phải các DB instance của nó), chọn **Actions** > **Delete**, bỏ chọn **Create final snapshot**, rồi gõ `delete me`.
{{% /notice %}}

#### (Tuỳ chọn) AWS CLI

{{%expand "Tạo read replica bằng AWS CLI" %}}
```bash
aws rds create-db-instance-read-replica \
    --db-instance-identifier rdspg-lab-cluster-read \
    --source-db-cluster-identifier rdspg-lab-cluster \
    --db-instance-class db.t4g.micro \
    --db-subnet-group-name rdspg-lab-subnet-group \
    --vpc-security-group-ids <ID của rds-sg> \
    --no-publicly-accessible
```
{{% /expand %}}
