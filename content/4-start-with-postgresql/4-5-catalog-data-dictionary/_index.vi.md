---
title : "Catalog và Data Dictionary"
weight : 5
chapter : false
pre : " <b> 4.5. </b> "
---

#### Tổng quan

PostgreSQL lưu thông tin về chính các đối tượng của nó, như bảng, cột, role và phiên kết nối, trong các bảng và view hệ thống. Tập hợp này được gọi là **system catalog** hay **data dictionary**.

| Nguồn | Chuẩn | Mô tả |
|---|---|---|
| `pg_catalog` | Riêng PostgreSQL | Các bảng và view hệ thống, đều có tiền tố `pg_`. Database nào cũng có schema này |
| `information_schema` | Chuẩn SQL | Các view mô tả đối tượng database theo cùng một cách trên PostgreSQL, MySQL, SQL Server và các hệ quản trị khác |

#### Xem catalog trong pgAdmin

1. Trong **Object Explorer**, mở **first_database** > **Catalogs** > **PostgreSQL Catalog (pg_catalog)**. pgAdmin nhóm các đối tượng catalog theo loại.

    ![PostgreSQL Catalog](/images/4-postgresql/4.5/1.png)

#### Truy vấn catalog

Chạy từng câu lệnh trong Query Tool của **first_database**: (1) nhập câu lệnh, (2) chọn **Execute**.

2. Liệt kê các bảng hệ thống trong `pg_catalog`:

    ```sql
    SELECT tablename FROM pg_tables WHERE schemaname = 'pg_catalog' ORDER BY tablename;
    ```

    ![pg_catalog tables](/images/4-postgresql/4.5/2.png)

3. Xem các phiên kết nối hiện tại bằng view `pg_stat_activity`:

    ```sql
    SELECT pid, usename, datname, application_name, state FROM pg_stat_activity WHERE datname IS NOT NULL;
    ```

    Kết quả hiển thị các phiên pgAdmin của bạn với user `postgres`, và các phiên khác đang mở, ví dụ `first_user` từ `psql`. Các phiên `rdsadmin` là kết nối nội bộ Amazon RDS dùng để quản lý instance.

    ![pg_stat_activity](/images/4-postgresql/4.5/3.png)

4. Liệt kê các bảng trong `first_schema` bằng `information_schema` theo chuẩn SQL:

    ```sql
    SELECT table_schema, table_name, table_type FROM information_schema.tables WHERE table_schema = 'first_schema';
    ```

    ![information_schema.tables](/images/4-postgresql/4.5/4.png)

{{% notice info %}}
`pg_stat_activity` thuộc **cumulative statistics system**. Từ PostgreSQL 15, các số liệu thống kê này được lưu trong shared memory thay vì qua một tiến trình statistics collector riêng.
{{% /notice %}}

{{% notice tip %}}
Để chẩn đoán hiệu năng trên Amazon RDS, hãy dùng **CloudWatch Database Insights**, tính năng bạn đã giữ ở mức Standard ở bước 2.7. Nó hiển thị tải database và wait event ở hiện tại lẫn trong quá khứ.
{{% /notice %}}
