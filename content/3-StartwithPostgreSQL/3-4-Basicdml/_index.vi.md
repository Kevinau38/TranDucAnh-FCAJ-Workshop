 ---
title : "Basic DML (Data Manipulation Language)"
date :  "`r Sys.Date()`" 
weight : 4 
chapter : false
pre : " <b> 3.4. </b> "
---

 Chương này sẽ nói về **truy vấn và DML cơ bản**.
 

#### Chèn dữ liệu vào bảng
 1. Trong pgAdmin tool, nhấp chuột phải vào **first_table** bên dưới **first_schema** bên trong **first_database** của bạn. Chọn **Scripts**, sau đó nhấp vào **INSERT Script**
 ![insert data](/images/2/2-4/16.png)
 Điều này sẽ tạo ra một câu lệnh chèn mẫu soạn sẵn cho bảng mà chúng ta có thể chỉnh sửa.\

 2. Làm nổi bật ? giữ chỗ và thay thế nó bằng `'hàng đầu tiên của tôi'`. Đừng quên sử dụng dấu ngoặc đơn `'` để bao quanh văn bản của bạn.
 ![insert data](/images/2/2-4/17.png)

 
 3. Nhấp vào biểu tượng Play để thực hiện lệnh này.
 Lệnh này sẽ chạy lệnh của bạn và chèn hàng đầu tiên của bạn.
 ![insert data](/images/2/2-4/18.png)
 {{%expand "'CHÈN 0 1' nghĩa là gì?" %}}Số 1 trong INSERT 0 1 có nghĩa là nó đã chèn 1 hàng. Số 0 chỉ áp dụng được nếu bảng sử dụng OID PostgreSQL cổ điển (Mã định danh đối tượng) và nếu vậy thì số 0 đại diện cho OID thực tế được tạo. Nếu không, nó sẽ luôn trả về giá trị 0. Ngày nay, bạn có thể sẽ không tạo bảng bằng OID, nhưng nếu tò mò, bạn có thể đọc thêm về OID trong tài liệu.{{% /expand%}}
 {{%expand "Hàng của tôi có được cam kết không?" %}}
 Theo mặc định, pgAdmin được đặt ở chế độ bật AutoCommit, vì vậy, hàng mới này phải được cam kết. Bạn có thể xem/thay đổi chế độ AutoCommit bằng cách nhấp vào biểu tượng bên cạnh biểu tượng Play trong Trình soạn thảo truy vấn pgAdmin:
 {{% /expand%}}

#### Truy vấn bảng đầu tiên của chúng tôi

 1. Trong pgAdmin của bạn, nhấp chuột phải vào **first_table** bên dưới **first_schema** bên trong **first_database** của bạn. Chọn Tập lệnh, sau đó nhấp vào **CHỌN Scripts**
 ![select](/images/2/2-4/19.png)

 2. Nhấp vào biểu tượng Play để thực hiện lệnh này.
  ![select](/images/2/2-4/20.png)
 Lệnh này sẽ chạy lệnh của bạn và bạn sẽ thấy hàng đầu tiên của mình.
 ![select](/images/2/2-4/21.png)


#### Tạo bảng bảng thứ hai nếu chưa tồn tại
 1. Sử dụng một trong các trình soạn thảo truy vấn, dán lệnh tạo sau đây rồi nhấp vào biểu tượng Play để thực thi lệnh đó:
    ``` 
    create table if not exists first_schema.second_table(
        col_pk serial,
        col_money numeric(12,2),
        col_short_str varchar(100),
        col_datetime timestamp(0) without time zone
      );
    ```
    ![add table](/images/2/2-4/22.png)

#### Chèn vào bảng thứ hai
 1. Sử dụng một trong các trình soạn thảo truy vấn mở, dán lệnh chèn sau rồi nhấp vào biểu tượng Play để thực thi lệnh đó:

    ```
    INSERT INTO first_schema.second_table(
      col_money, col_short_str, col_datetime)
      VALUES (40.25, 'short string', TIMESTAMP '2004-10-19 10:23:54') 
    returning col_pk;
    ```
    ![insert data to table](/images/2/2-4/23.png)

 Lưu ý rằng vì chúng ta đã chỉ định `returning col_pk`, nên chúng ta thấy giá trị duy nhất được tạo tự động của col_pk trong phần Data Output. Hãy nhớ rằng khi tạo bảng thứ hai, chúng ta đã chỉ định rằng col_pk đã sử dụng kiểu dữ liệu `SERIAL` (giống như kiểu dữ liệu AUTO_INCREMENT trong các công cụ cơ sở dữ liệu khác).

#### Thử nghiệm chuyển đổi kiểu dữ liệu
 1. Sử dụng trình soạn thảo truy vấn đang mở, dán lệnh chọn sau và nhấp vào biểu tượng Play để thực thi lệnh đó:
    ```
    SELECT '40.25' col1, 40.25 col2;
    ```
    ![datatype](/images/2/2-4/24.png)
  Lưu ý trong phần Data Output  `col1` là kiểu dữ liệu text  trong khi `col2` là kiểu dữ liệu numeric .
  {{% notice note%}}
  Không giống như một số cơ sở dữ liệu khác (cụ thể là Oracle), bạn không cần phải có mệnh đề FROM trong câu lệnh SELECT của mình. Vì vậy, nếu bạn có thói quen viết `SELECT 'something' FROM dual` trong Oracle, bạn có thể tắt `FROM Dual` trong PostgreSQL. Không có bảng `dual` trong PostgreSQL.
  {{% /notice %}}

 2. Bây giờ hãy chạy truy vấn này:

    ```
    SELECT '40.25' col1, 40.25 col2, '40.25'::numeric col3, numeric '40.25' col4, cast('40.25' as numeric) col5;
    ```
    ![datatype](/images/2/2-4/25.png)

 Lưu ý trong phần Dữ liệu đầu ra `col3`, `col4` và `col5` đều là ví dụ về kiểu dữ liệu text đã được chuyển đổi thành kiểu dữ liệu numeric . 3 cột này trình bày 3 cách mà bạn có thể thực hiện chuyển đổi kiểu dữ liệu trong câu lệnh SQL PostgreSQL.