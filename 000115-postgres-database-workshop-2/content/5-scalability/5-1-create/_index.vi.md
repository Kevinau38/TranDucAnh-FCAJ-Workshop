---
title : "Tạo read replica"
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Tổng quan

**Read replica** là bản sao chỉ đọc của một DB instance. RDS tạo nó từ snapshot của nguồn, rồi giữ cho nó luôn cập nhật bằng **streaming replication bất đồng bộ** của PostgreSQL. Ứng dụng có thể gửi các truy vấn chỉ đọc (báo cáo, dashboard, tìm kiếm) tới replica để primary rảnh cho việc ghi.

| | Primary | Read replica |
|---|---|---|
| Đọc | Có | Có |
| Ghi | Có | Không, replica chỉ đọc |
| Endpoint | Riêng | Riêng, khác với primary |
| Dữ liệu | Mới nhất | Chậm vài giây (**replica lag**) |
| Số lượng | 1 | Tối đa 15 cho mỗi nguồn, cùng hoặc khác Region |

#### Tạo read replica

1. Trong [danh sách cơ sở dữ liệu](https://console.aws.amazon.com/rds/home#databases:):
    - (1) Chọn `rdspg-lab-db`.
    - (2) Chọn **Actions**.
    - (3) Chọn **Create read replica**.

    ![Create read replica](/images/5/5-1/1.png)

2. Trong **Settings** và **Instance configuration**:
    - (1) **Replica source** là `rdspg-lab-db`.
    - (2) **DB instance identifier**: `rdspg-lab-db-read`
    - (3) **Burstable classes**, **Instance type** `db.t4g.micro`

    ![Settings](/images/5/5-1/2.png)

{{% notice tip %}}
Replica có thể dùng instance class khác primary. Hãy giữ class bằng hoặc lớn hơn primary cho replica mà bạn có thể sẽ promote, hoặc replica phải theo kịp một primary bận rộn.
{{% /notice %}}

3. Trong **Availability**, console mặc định chọn **Multi-AZ DB instance deployment**:
    - (1) Chọn **Single-AZ DB instance deployment (1 instance)** để giữ chi phí thấp.

    ![Availability](/images/5/5-1/3.png)

4. Trong **Connectivity**, console chép cấu hình từ nguồn. Hãy kiểm tra:
    - (1) **DB subnet group**: `rdspg-lab-subnet-group`
    - (2) **Public access**: **Not publicly accessible**
    - (3) **Existing VPC security groups**: `rds-sg`

    ![Connectivity](/images/5/5-1/4.png)

    Giữ các cấu hình còn lại và chọn **Create read replica** ở cuối trang.

5. Trong danh sách cơ sở dữ liệu:
    - (1) Trạng thái của `rdspg-lab-db` chuyển sang **Modifying** trong lúc RDS chụp snapshot. Vai trò của nó giờ là **Primary**.
    - (2) `rdspg-lab-db-read` xuất hiện bên dưới với vai trò **Replica** và trạng thái **Creating**.

    ![Creating](/images/5/5-1/5.png)

    Replica mất khoảng 10 đến 15 phút. `rdspg-lab-db` vẫn hoạt động suốt thời gian này.

#### Xem replica

6. Khi `rdspg-lab-db-read` ở trạng thái **Available**, chọn nó. Trên tab **Connectivity & security**, chọn **Endpoints** trong **Connect using** và mở **Additional configurations**:
    - (1) Replica có **Endpoint** riêng, khác với endpoint của `rdspg-lab-db`.
    - (2) Nó dùng cùng security group `rds-sg`.

    ![Replica endpoint](/images/5/5-1/6.png)

7. Kéo xuống **Replication**:
    - (1) `rdspg-lab-db-read` có vai trò **Replica**, nguồn `rdspg-lab-db`, trạng thái **Replicating** và độ trễ vài giây.

    ![Replication](/images/5/5-1/7.png)

#### Kết nối vào replica

8. Trên App server, kết nối tới endpoint của replica. Nhập mật khẩu master khi `psql` hỏi:

    ```bash
    R=<endpoint của rdspg-lab-db-read>
    psql -h $R -d pglab \
        -c "SELECT pg_is_in_recovery() AS is_replica, count(*) AS accounts FROM pgbench_accounts;" \
        -c "SELECT extract(epoch from now() - pg_last_xact_replay_timestamp()) AS replica_lag_seconds;" \
        -c "CREATE TABLE test_write(id int);"
    ```

    - (1) `pg_is_in_recovery()` là `t`: máy chủ là replica. Nó có đủ 5.000.000 dòng của `pgbench_accounts` từ chương 3.
    - (2) `replica_lag_seconds` là thời gian kể từ giao dịch cuối cùng replica đã áp dụng.
    - (3) Replica từ chối ghi: *cannot execute CREATE TABLE in a read-only transaction*.

    ![psql on the replica](/images/5/5-1/8.png)

{{% notice note %}}
`replica_lag_seconds` tăng dần khi không ai ghi vào primary, vì nó đo thời gian kể từ giao dịch **cuối cùng** được áp dụng. Nó chỉ phản ánh đúng độ trễ khi primary đang bận. Metric CloudWatch `ReplicaLag` của replica đáng tin cậy hơn.
{{% /notice %}}

#### (Tùy chọn) AWS CLI

{{%expand "Tạo read replica bằng AWS CLI" %}}
```bash
aws rds create-db-instance-read-replica \
    --db-instance-identifier rdspg-lab-db-read \
    --source-db-instance-identifier rdspg-lab-db \
    --db-instance-class db.t4g.micro \
    --no-multi-az \
    --no-publicly-accessible

aws rds wait db-instance-available --db-instance-identifier rdspg-lab-db-read
```
{{% /expand %}}
