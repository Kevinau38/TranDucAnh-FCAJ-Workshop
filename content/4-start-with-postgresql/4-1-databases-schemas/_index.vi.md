---
title : "Database và Schema"
weight : 1
chapter : false
pre : " <b> 4.1. </b> "
---

#### Tổng quan

PostgreSQL tổ chức đối tượng theo bốn tầng:

```
DB instance (rdspg-lab-db) > Database > Schema > Table
```

| Tầng | Mô tả |
|---|---|
| DB instance | Còn gọi là PostgreSQL **cluster**. Role (user) dùng chung cho mọi database trong instance |
| Database | Dữ liệu giữa các database không dùng chung. Mỗi kết nối chỉ làm việc với một database tại một thời điểm |
| Schema | Không gian tên bên trong database, giống một thư mục. Hai schema có thể chứa bảng trùng tên |

Mỗi database có sẵn schema mặc định tên `public`. Khác với MySQL, nơi database và schema gần như là một, PostgreSQL tách chúng thành hai tầng riêng.

{{% notice info %}}
Chương này giả định pgAdmin 4 đã kết nối tới `rdspg-lab-db` như ở bước 3.2.
{{% /notice %}}

#### Tạo database

1. Trong **Object Explorer**, mở **rdspg-lab-db**:
    - (1) Chuột phải **Databases**.
    - (2) Chọn **Create**.
    - (3) Chọn **Database...**.

    ![Create database](/images/4-postgresql/4.1/1.png)

2. Trong tab **General**:
    - (1) **Database**: `first_database`. Giữ **Owner** là `postgres`.

    ![Database name](/images/4-postgresql/4.1/2.png)

3. Xem câu lệnh pgAdmin sẽ chạy:
    - (1) Chọn tab **SQL**.
    - (2) Chọn **Save**.

    ![CREATE DATABASE statement](/images/4-postgresql/4.1/3.png)

{{% notice note %}}
Mọi thao tác trong pgAdmin đều chạy một câu lệnh SQL. Tab **SQL** hiển thị câu lệnh đó trước khi lưu, giúp bạn học được SQL đằng sau mỗi thao tác.
{{% /notice %}}

4. Mở **first_database** > **Schemas**. Database mới đã có sẵn schema `public`.

    ![first_database and public schema](/images/4-postgresql/4.1/4.png)

#### Tạo schema

5. Trong **first_database**:
    - (1) Chuột phải **Schemas**.
    - (2) Chọn **Create**.
    - (3) Chọn **Schema...**.

    ![Create schema](/images/4-postgresql/4.1/5.png)

6. Trong tab **General**:
    - (1) **Name**: `first_schema`. Giữ **Owner** là `postgres`.
    - (2) Chọn **Save**.

    ![Schema name](/images/4-postgresql/4.1/6.png)

7. `first_schema` xuất hiện trong **Schemas**.

    ![first_schema](/images/4-postgresql/4.1/7.png)

{{% notice tip %}}
Khi câu truy vấn dùng tên bảng không kèm schema, PostgreSQL tìm bảng trong các schema thuộc `search_path`, mặc định là `"$user", public`. Bảng nằm trong `first_schema` phải viết dạng `first_schema.ten_bang`, trừ khi bạn đổi `search_path`. Bạn sẽ thực hành phần này ở bước 4.6.
{{% /notice %}}
