---
title : "Roles và Users"
date :  "`r Sys.Date()`" 
weight : 5 
chapter : false
pre : " <b> 3.5. </b> "
---
 Chương này sẽ nói về [PostgreSQL Roles](https://www.postgresql.org/docs/11/user-manag.html) . Trong PostgreSQL, Roles bao hàm khái niệm về cả người dùng và nhóm. Các câu lệnh CREATE USER và CREATE GROUP thực chất là bí danh của câu lệnh CREATE ROLE. Sự khác biệt là người dùng có vai trò với đặc quyền **LOGIN**.

 Roles được xác định ở cấp cụm PostgreSQL và có giá trị trong tất cả các cơ sở dữ liệu trong cụm. Roles không được tạo cụ thể cho một cơ sở dữ liệu riêng lẻ.
 
 
 {{% notice info%}}
Trong PostgreSQL, schemas là khái niệm khác biệt với roles/users. Điều này khác với cơ sở dữ liệu như Oracle. {{% /notice %}}
 #### Exploring Existing Roles

 Trong pgAdmin, hãy mở rộng mục **Login/Group Roles** trong PostgreSQL server.

  ![role](/images/2/2-5/26.png)

  Bạn sẽ thấy các [default roles][https://www.postgresql.org/docs/11/default-roles.html) đã có và được xác định trong cụm PostgreSQL này. Lưu ý rằng có 2 loại biểu tượng mà pgAdmin sử dụng cho roles. Nếu roles có đặc quyền **LOIGN** thì roles đó sẽ được xác định là người dùng (chẳng hạn như rdsadmin và masteruser). Nếu không, nó sẽ được hiển thị với biểu tượng group.\

Bạn cũng có thể truy xuất thông tin tương tự bằng cách chạy truy vấn sql sau:

 ``` SELECT rolname FROM pg_roles;```

 ![role](/images/2/2-5/27.png)

#### Tạo Role

 1. Nhấp chuột phải vào **ĐLogin/Group Roles**, sau đó chọn **Create**, sau đó chọn **Login/Group role**:
  ![role](/images/2/2-5/28.png)

 2. Đặt tên cho role: ``first_role``.
  ![role](/images/2/2-5/28.png)

 3. Duyệt qua các tab khác của hộp thoại Create Role nhưng không thay đổi bất cứ điều gì. Kết thúc trên tab SQL. Sau đó nhấp vào **Save**.
 ![role](/images/2/2-5/30.png)

 4. Nhấp chuột phải vào **first_database** và chọn biểu tượng **Query Tool**.
 ![role](/images/2/2-5/31.png)
 5. Dán và chạy câu lệnh SQL sau để cấp một số [đặc quyền](https://www.postgresql.org/docs/11/ddl-priv.html) cho vai trò mới của bạn.
    ```
    GRANT USAGE ON SCHEMA first_schema TO first_role;
    GRANT SELECT ON first_schema.first_table TO first_role;
    ```
    ![role](/images/2/2-5/32.png)

 
 {{% notice note%}}
 Chúng tôi đã cấp cho vai trò cả khả năng SELECT từ bảng và cả khả năng sử dụng schemas chứa bảng.
 {{% /notice %}}

#### Tạo người dùng mới

 1. Nhấp chuột phải vào **Login/Group Roles**, sau đó chọn **Create**, sau đó chọn **Login/Group role**:
    ![role](/images/2/2-5/33.png)

 2. Đặt tên cho role ``first_user``.
    ![role](/images/2/2-5/34.png)

 3. Click on the **Definition** tab. Then enter a Password you will remember.
    ![role](/images/2/2-5/35.png)

 4. Nhấp vào tab **Privileges**. Sau đó đặt **Can login?** thành **Yes**
    ![role](/images/2/2-5/36.png)

 5. Nhấp vào tab **Membership**. Sau đó nhấp vào trường Roles và chọn ``first_role``
    ![role](/images/2/2-5/37.png)

 6. Cuối cùng, nhấp vào tab **SQL** để xem **SQL** trông như thế nào. Sau đó **Save**
    ![role](/images/2/2-5/38.png)

 Bây giờ, nhấp chuột phải vào nút **Servers** và chọn **Create**, sau đó nhấp vào **Server**

 7. Đặt tên cho kết nối mới ``first_user_connection``\
 8. Nhấp vào tab **Connection**. Nhập RDS endpoint cho tên máy chủ (xem ghi chú bên dưới nếu bạn cần trợ giúp). Nhập ``first_user`` cho Tên người dùng. Nhập **Mật khẩu** cho người dùng mới. Sau đó nhấp vào **Save**.
  ![role](/images/2/2-5/39.png)

 9. Mở rộng nút **first_user_connection**, sau đó mở rộng **Databases**. Nhấp chuột phải vào **first_database**, sau đó chọn **Query Tool** để mở phiên với tư cách first_user.
   ![role](/images/2/2-5/40.png)

 10. Dán và chạy câu lệnh SQL này
    ```
    select * from first_schema.first_table;
    ```
    ![role](/images/2/2-5/41.png)

 11. Dán và chạy câu lệnh SQL này để kiểm tra bảng mà chúng ta chưa cấp cho **first_role**:
    ```select * from first_schema.second_table;```
    ![role](/images/2/2-5/42.png)
  {{% notice tip%}}
 Đọc thêm về [các phương pháp hay nhất để quản lý User/Role dùng PostgreSQL](https://aws.amazon.com/blogs/database/managing-postgresql-users-and-roles/)
 {{% /notice %}}

{{%expand " Bạn muốn tạo thêm người dùng DBA (tương tự như người dùng chính) cho phiên bản RDS hoặc Aurora PostgreSQL của mình? " %}}
    Hãy thử [tập lệnh này](https://aws.amazon.com/premiumsupport/know-center/rds-aurora-postgresql-clone-master-user/):
    ```
    --replace thepassword with a real password
    CREATE ROLE new_dba_user WITH PASSWORD 'thepassword' CREATEDB CREATEROLE LOGIN;
    GRANT rds_superuser TO new_dba_user;

    ```
    {{% /expand%}}    