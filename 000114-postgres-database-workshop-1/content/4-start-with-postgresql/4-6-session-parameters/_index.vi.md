---
title : "Session Parameter"
weight : 6
chapter : false
pre : " <b> 4.6. </b> "
---

#### Tổng quan

Hoạt động của PostgreSQL được điều khiển bằng các **parameter**. Một số parameter áp dụng cho toàn bộ DB instance, một số có thể thay đổi riêng cho từng **session** (một kết nối).

| Phạm vi | Cách thay đổi | Hiệu lực |
|---|---|---|
| Toàn bộ DB instance | Sửa **DB parameter group** trong Amazon RDS, ví dụ `default.postgres18` ở bước 2.7 | Ngay lập tức hoặc sau khi reboot, tùy parameter |
| Một session | Chạy `SET` trong session | Ngay lập tức, chỉ cho kết nối hiện tại |

#### Xem parameter

Chạy từng câu lệnh trong Query Tool của **first_database**: (1) nhập câu lệnh, (2) chọn **Execute**.

1. Liệt kê các parameter mà user có thể thay đổi cho session:

    ```sql
    SELECT name, setting, context FROM pg_settings WHERE context = 'user' ORDER BY name;
    ```

    ![User-level parameters](/images/4-postgresql/4.6/1.png)

2. Xem giá trị hiện tại của `search_path`:

    ```sql
    SHOW search_path;
    ```

    Giá trị mặc định là `"$user", public`. PostgreSQL tìm trước trong schema trùng tên với user hiện tại, sau đó tới `public`.

    ![SHOW search_path](/images/4-postgresql/4.6/2.png)

#### Thay đổi session parameter

3. Truy vấn `first_table` mà không ghi tên schema:

    ```sql
    SELECT * FROM first_table;
    ```

    Câu truy vấn báo lỗi `relation "first_table" does not exist` vì `first_schema` không nằm trong `search_path`.

    ![Table not found](/images/4-postgresql/4.6/3.png)

4. Thêm `first_schema` vào `search_path` cho session này, rồi chạy lại câu truy vấn:

    ```sql
    SET search_path TO first_schema, public;
    SELECT * FROM first_table;
    ```

    Câu truy vấn trả về `my first row`.

    ![SET search_path](/images/4-postgresql/4.6/4.png)

{{% notice note %}}
`SET` chỉ thay đổi giá trị cho session hiện tại. Các kết nối khác và kết nối mới vẫn dùng giá trị mặc định. Chạy `RESET search_path;` để đưa về giá trị mặc định trong session hiện tại.
{{% /notice %}}

{{% notice tip %}}
Sau khi sửa parameter trong DB parameter group, hãy kiểm tra giá trị đang có hiệu lực bằng `SHOW` hoặc `pg_settings`. Một số parameter chỉ thay đổi sau khi DB instance reboot, nên giá trị trong parameter group và giá trị đang có hiệu lực có thể khác nhau.
{{% /notice %}}
