---
title : "Kiểm tra kết quả nâng cấp"
weight : 3
chapter : false
pre : " <b> 2.3. </b> "
---

#### Tổng quan

Sau khi nâng cấp phiên bản chính, hãy kiểm tra kết quả ở ba nơi: các sự kiện của DB instance, cấu hình của nó và chính cơ sở dữ liệu. Sau đó cập nhật lại thống kê cho bộ tối ưu truy vấn, vì `pg_upgrade` không chép thống kê sang phiên bản mới.

#### Kiểm tra trên console

1. Khi **Status** của `rdspg-lab-db` là **Available**, mở tab **Logs & events**:
    - (1) Sự kiện *Database instance engine major version upgrade complete. Previous version: 17.11.R1. New version: 18.6.R1* cũng cho thấy parameter group đã đổi.

    ![Major version upgrade event](/images/2/2-3/1.png)

2. Mở tab **Configuration**:
    - (1) **Engine version** là **18.6**.
    - (2) **Option groups** là `default:postgres-18`, trạng thái **In sync**.

    ![Configuration](/images/2/2-3/2.png)

#### Kiểm tra từ App server

3. Kết nối vào App server (`ssh rdspg-app`), rồi chạy:

    ```bash
    psql -d postgres -c "SELECT version();"
    vacuumdb -d postgres --analyze-in-stages
    vacuumdb -d pglab --analyze-in-stages
    ```

    - (1) Phiên bản máy chủ là **PostgreSQL 18.6**.
    - (2) `vacuumdb --analyze-in-stages` dựng lại thống kê của từng cơ sở dữ liệu, đầu tiên nhanh với ít mẫu, sau đó với độ chính xác đầy đủ.

    ![Version and statistics](/images/2/2-3/3.png)

{{% notice warning %}}
Không chạy `vacuumdb --all` trên Amazon RDS. Lệnh này cũng cố kết nối vào `rdsadmin`, cơ sở dữ liệu nội bộ chỉ AWS được dùng, và báo lỗi *pg_hba.conf rejects connection*. Hãy chạy `vacuumdb` cho từng cơ sở dữ liệu của bạn.
{{% /notice %}}

{{% notice tip %}}
Sau khi nâng cấp phiên bản chính, hãy cập nhật cả các extension bạn dùng bằng `ALTER EXTENSION ... UPDATE;`. RDS không tự cập nhật chúng.
{{% /notice %}}

#### (Tùy chọn) AWS CLI

{{%expand "Kiểm tra phiên bản engine bằng AWS CLI" %}}
```bash
aws rds describe-db-instances \
    --db-instance-identifier rdspg-lab-db \
    --query 'DBInstances[0].[EngineVersion,DBParameterGroups[0].DBParameterGroupName]' \
    --output table
```
{{% /expand %}}
