---
title : "Table và kiểu dữ liệu"
weight : 2
chapter : false
pre : " <b> 4.2. </b> "
---

#### Tổng quan

**Table** (bảng) là tập hợp các dòng có tên. Mọi dòng có cùng các cột, và mỗi cột có một **kiểu dữ liệu** (data type) xác định. Tài liệu PostgreSQL còn gọi table là **relation**, thuật ngữ toán học đứng sau cụm từ "relational database".

SQL không đảm bảo thứ tự các dòng trong bảng. Muốn lấy dữ liệu theo thứ tự, hãy dùng `ORDER BY` trong câu truy vấn.

#### Tạo bảng

1. Trong **Object Explorer**, tại **first_database** > **Schemas**:
    - (1) Chuột phải **first_schema**.
    - (2) Chọn **Create**.
    - (3) Chọn **Table...**.

    ![Create table](/images/4-postgresql/4.2/1.png)

2. Trong tab **General**:
    - (1) **Name**: `first_table`
    - (2) **Schema**: `first_schema`

    ![Table name](/images/4-postgresql/4.2/2.png)

3. Trong tab **Columns**:
    - (1) Chọn tab **Columns**.
    - (2) Chọn **+** để thêm cột.
    - (3) **Name**: `col1`
    - (4) **Data type**: `text`
    - (5) Bật **Not NULL?**.
    - (6) Bật **Primary key?**.

    ![Columns](/images/4-postgresql/4.2/3.png)

4. Xem câu lệnh và lưu:
    - (1) Chọn tab **SQL**. pgAdmin tạo bảng bằng `CREATE TABLE first_schema.first_table`.
    - (2) Chọn **Save**.

    ![CREATE TABLE statement](/images/4-postgresql/4.2/4.png)

5. Mở **first_table** > **Columns** để thấy cột `col1` với kiểu dữ liệu `text`.

    ![first_table](/images/4-postgresql/4.2/5.png)

{{% notice note %}}
Tab **General** còn có **Partitioned table?**, và tab **Columns** có **Inherited from table(s)**. Workshop này không dùng partitioning hay table inheritance, nhưng đây là hai tính năng hữu ích của PostgreSQL cho bảng lớn hoặc các bảng có liên quan với nhau.
{{% /notice %}}

#### Các kiểu dữ liệu thường dùng

| Nhóm | Kiểu dữ liệu | Khi nào dùng |
|---|---|---|
| Số nguyên | `integer`, `bigint` | `integer` là lựa chọn phổ biến. Dùng `bigint` cho giá trị rất lớn |
| Số thập phân chính xác | `numeric(p, s)` | Tiền tệ và các giá trị cần chính xác tuyệt đối |
| Tự tăng | `serial`, `bigserial` hoặc `GENERATED ... AS IDENTITY` | Cột ID duy nhất. Identity column là cách theo chuẩn SQL |
| Chuỗi | `text`, `varchar(n)` | `text` không giới hạn độ dài. `varchar(n)` giới hạn độ dài. Hiệu năng hai kiểu như nhau |
| Ngày giờ | `date`, `time`, `timestamp`, `timestamptz` | `timestamptz` lưu thời điểm kèm múi giờ |
| Logic | `boolean` | `true` hoặc `false` |
| JSON | `json`, `jsonb` | `jsonb` lưu dạng nhị phân và hỗ trợ đánh index |

{{% notice tip %}}
Hạn chế dùng `char(n)` trong PostgreSQL. Kiểu này tự thêm khoảng trắng và thường chậm hơn `text` hoặc `varchar(n)`. Xem [Data Types](https://www.postgresql.org/docs/current/datatype.html) trong tài liệu PostgreSQL để biết danh sách đầy đủ.
{{% /notice %}}
