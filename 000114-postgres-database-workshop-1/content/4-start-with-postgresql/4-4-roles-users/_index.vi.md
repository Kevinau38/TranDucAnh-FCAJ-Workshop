---
title : "Role và User"
weight : 4
chapter : false
pre : " <b> 4.4. </b> "
---

#### Tổng quan

Trong PostgreSQL, **role** bao gồm cả user và group. **User** đơn giản là một role có thuộc tính `LOGIN`. `CREATE USER` và `CREATE GROUP` đều là tên gọi khác của `CREATE ROLE`.

Role được định nghĩa ở cấp DB instance và có hiệu lực trong mọi database của instance. Schema và role là hai khái niệm tách biệt, khác với Oracle nơi schema gắn với một user.

Ở bước này, bạn tạo:

| Role | Đăng nhập | Quyền |
|---|---|---|
| `first_role` | Không | Chỉ đọc `first_schema.first_table` |
| `first_user` | Có | Thành viên của `first_role`, nên thừa hưởng cùng quyền |

#### Xem các role có sẵn

1. Trong Query Tool của **first_database**, chạy:
    - (1) Nhập câu lệnh.
    - (2) Chọn **Execute**.

    ```sql
    SELECT rolname, rolsuper, rolcanlogin FROM pg_roles
    WHERE rolname NOT LIKE 'pg\_%' ORDER BY rolname;
    ```

    ![Existing roles](/images/4-postgresql/4.4/1.png)

| Role | Mô tả |
|---|---|
| `postgres` | Master user. `rolsuper` là `false` vì Amazon RDS không cấp superuser thật cho bạn |
| `rds_superuser` | Role cấp quyền quản trị cho master user |
| `rdsadmin` | Superuser thật mà Amazon RDS dùng để quản lý instance |

#### Tạo role và user

2. Thay bằng khối lệnh sau và chạy:
    - (1) Nhập các câu lệnh.
    - (2) Chọn **Execute**.

    ```sql
    CREATE ROLE first_role;
    GRANT USAGE ON SCHEMA first_schema TO first_role;
    GRANT SELECT ON first_schema.first_table TO first_role;

    CREATE ROLE first_user WITH LOGIN PASSWORD 'FirstUser#2026';
    GRANT first_role TO first_user;
    ```

    ![Create role and user](/images/4-postgresql/4.4/2.png)

{{% notice note %}}
Để đọc một bảng trong schema, role cần hai quyền: `USAGE` trên schema và `SELECT` trên bảng. Thiếu `USAGE` thì role không truy cập được đối tượng nào trong schema.
{{% /notice %}}

3. Trong **Object Explorer**, chuột phải **Login/Group Roles** và chọn **Refresh**. `first_role` hiển thị với biểu tượng nhóm, còn `first_user` hiển thị biểu tượng người vì đăng nhập được.

    ![Roles in Object Explorer](/images/4-postgresql/4.4/3.png)

#### Kiểm tra quyền

4. Kết nối tới App server (`ssh rdspg-app`) và đăng nhập bằng `first_user`. Nhập `FirstUser#2026` khi được hỏi.

    ```bash
    export RDSHOST="<RDS_ENDPOINT>"
    psql "host=$RDSHOST port=5432 dbname=first_database user=first_user sslmode=verify-full sslrootcert=./global-bundle.pem"
    ```

    Chạy từng câu truy vấn trên một dòng riêng:

    ```sql
    SELECT * FROM first_schema.first_table;
    ```

    ```sql
    SELECT * FROM first_schema.second_table;
    ```

    `first_user` đọc được `first_table`, nhưng bị `permission denied` với `second_table` vì `first_role` không có quyền trên bảng đó.

    ![Test privileges](/images/4-postgresql/4.4/4.png)

5. Gõ `\q` để thoát `psql`.

{{% notice tip %}}
Hãy cấp quyền cho group role, rồi thêm user vào role đó. Khi một người đổi vị trí công việc, bạn chỉ cần đổi role của họ thay vì sửa quyền trên từng bảng. Xem [Managing PostgreSQL users and roles](https://aws.amazon.com/blogs/database/managing-postgresql-users-and-roles/) để biết các best practice trên Amazon RDS.
{{% /notice %}}
