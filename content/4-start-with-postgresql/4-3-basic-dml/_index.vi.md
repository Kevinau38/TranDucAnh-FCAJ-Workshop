---
title : "DML cơ bản (Data Manipulation Language)"
weight : 3
chapter : false
pre : " <b> 4.3. </b> "
---

#### Tổng quan

**DML** (Data Manipulation Language) là nhóm câu lệnh SQL làm việc với dữ liệu trong bảng: `INSERT`, `SELECT`, `UPDATE` và `DELETE`. Ở bước này, bạn chạy SQL trực tiếp trong **Query Tool** của pgAdmin.

{{% notice info %}}
Bước này dùng bảng `first_schema.first_table` đã tạo ở bước 4.2.
{{% /notice %}}

#### Mở Query Tool

1. Trong **Object Explorer**:
    - (1) Chuột phải **first_database**.
    - (2) Chọn **Query Tool**.

    ![Query Tool](/images/4-postgresql/4.3/1.png)

{{% notice tip %}}
Bạn cũng có thể mở Query Tool bằng **Alt + Shift + Q**. Query Tool kết nối tới database đang được chọn, hiển thị ở thanh phía trên vùng soạn lệnh.
{{% /notice %}}

#### Thêm và truy vấn dữ liệu

2. Thêm dòng đầu tiên:
    - (1) Nhập câu lệnh sau.
    - (2) Chọn **Execute** (hoặc nhấn **F5**).

    ```sql
    INSERT INTO first_schema.first_table (col1) VALUES ('my first row');
    ```

    Tab **Messages** hiển thị `INSERT 0 1`. Số `1` là số dòng đã thêm.

    ![INSERT](/images/4-postgresql/4.3/2.png)

{{% notice note %}}
pgAdmin mặc định bật **auto-commit**, nên dòng mới được commit ngay.
{{% /notice %}}

3. Thay câu lệnh và chạy:
    - (1) Nhập câu lệnh sau.
    - (2) Chọn **Execute**.

    ```sql
    SELECT * FROM first_schema.first_table;
    ```

    ![SELECT](/images/4-postgresql/4.3/3.png)

#### Tạo bảng thứ hai và trả về giá trị tự sinh

4. Thay bằng khối lệnh sau và chạy:
    - (1) Nhập các câu lệnh.
    - (2) Chọn **Execute**.

    ```sql
    CREATE TABLE IF NOT EXISTS first_schema.second_table (
        col_pk        integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
        col_money     numeric(12,2),
        col_short_str varchar(100),
        col_datetime  timestamp(0)
    );

    INSERT INTO first_schema.second_table (col_money, col_short_str, col_datetime)
    VALUES (40.25, 'short string', TIMESTAMP '2026-10-29 10:23:54')
    RETURNING col_pk;
    ```

    `col_pk` là **identity column**, nên PostgreSQL tự sinh giá trị. `RETURNING col_pk` trả về giá trị vừa sinh là `1` mà không cần truy vấn thêm.

    ![RETURNING](/images/4-postgresql/4.3/4.png)

{{% notice info %}}
`GENERATED ALWAYS AS IDENTITY` là cách tạo cột tự tăng theo chuẩn SQL. Code PostgreSQL cũ thường dùng `serial`, hoạt động tương tự.
{{% /notice %}}

#### Chuyển đổi kiểu dữ liệu

5. Thay câu lệnh và chạy:
    - (1) Nhập câu lệnh sau.
    - (2) Chọn **Execute**.

    ```sql
    SELECT '40.25' AS col1, 40.25 AS col2, '40.25'::numeric AS col3, CAST('40.25' AS numeric) AS col4;
    ```

    **Data Output** hiển thị kiểu dữ liệu dưới tên mỗi cột. `col1` là `text` vì giá trị nằm trong dấu nháy. `col3` và `col4` chuyển chuỗi sang `numeric` bằng `::` và `CAST`.

    ![Type conversion](/images/4-postgresql/4.3/5.png)

{{% notice note %}}
Khác với Oracle, câu `SELECT` trong PostgreSQL không bắt buộc có mệnh đề `FROM`, và PostgreSQL không có bảng `dual`.
{{% /notice %}}
