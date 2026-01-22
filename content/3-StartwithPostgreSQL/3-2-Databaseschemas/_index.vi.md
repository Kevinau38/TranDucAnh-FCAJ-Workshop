---
title : "Database và Schemas"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 3.2. </b> "
---
Chào mừng đến với PostgreSQL! Nếu đây là lần đầu tiên bạn tìm hiểu về **PostgreSQL**, chúng tôi khuyến khích bạn xem trang web chính thức [Giới thiệu về PostgreSQL](https://www.postgresql.org/about/).

Trong module này, chúng ta sẽ tìm hiểu về **Databases và Schemas.**

{{% notice info %}}
Vui lòng hoàn thành việc cài đặt và cấu hình trước khi tiếp tục tìm hiểu về module này.
{{% /notice %}}

#### Tìm hiểu về Databases

 1. Trong pgAdmin tool, hãy nhấp vào *>* ở phía trước **rdspg-fcj-labs** để mở rộng nó.
      ![rdspg-fcj-labs](/images/2/2-2/1.png)
 {{% notice note %}}
 Bạn thấy 3 mục: Databases, Login/Group Roles, and Tablespaces.\
 Trong phần này, chúng ta sẽ tập trung vào **Databases**. Và chúng ta sẽ đề cập đến **Login/Group** trong phần sau.
 {{% /notice %}}

 
 2. Tìm hiểu về Databases node.
   ![Databases](/images/2/2-2/2.png)
     Theo thuật ngữ ngành, PostgreSQL instance `(rdspg-fcj-labs)` mà bạn đã tạo được gọi là **cụm** PostgreSQL. Một cụm chứa một hoặc nhiều **cơ sở dữ liệu**. Mặc dù users/roles của một cụm được chia sẻ nhưng không có dữ liệu nào được chia sẻ trên các cơ sở dữ liệu. Nói cách khác, khi khách hàng kết nối với một cụm, kết nối đó bắt buộc phải chỉ định cơ sở dữ liệu mà họ muốn làm việc và kết nối đó chỉ có thể hoạt động trong một cơ sở dữ liệu tại một thời điểm.\
 {{% notice note %}}
 bạn sẽ thấy một số cơ sở dữ liệu trong cụm `pglab`.
 {{% /notice %}}
 {{%expand "Cơ sở dữ liệu `rdsadmin` là gì?" %}}Cơ sở dữ liệu có tên rdsadmin là cơ sở dữ liệu được dành riêng để sử dụng bởi RDS/Aurora.{{% /expand%}}

 3. Nhấp chuột phải vào **Databases** và chọn **Create**, sau đó nhấp vào **Databases**
   ![Create Databases](/images/2/2-2/3.png)

 4. Đặt tên cơ sở dữ liệu là `first_database` (nhưng chưa lưu)
      ![My Databases](/images/2/2-2/4.png)
 {{% notice note %}}
 Mọi cơ sở dữ liệu đều có chủ sở hữu, đây là vai trò có toàn quyền kiểm soát cơ sở dữ liệu và có thể gán quyền cho các vai trò khác. Theo mặc định, khi cơ sở dữ liệu được tạo trong PostgreSQL, vai trò được sử dụng để tạo cơ sở dữ liệu sẽ trở thành chủ sở hữu của cơ sở dữ liệu đó. Trong PGAdmin, vai trò bạn đăng nhập thường sẽ được đặt làm chủ sở hữu cơ sở dữ liệu mà bạn hiện đang làm việc. Điều đáng nói là PostgreSQL cho phép bạn thay đổi chủ sở hữu của hầu hết các đối tượng cơ sở dữ liệu ngay cả sau khi chúng đã được tạo
 {{% /notice %}} 

 5. Hãy nhấp vào tab **Definition**
   ![Definition](/images/2/2-2/5.png)

{{% notice note %}}
 Có [nhiều cài đặt khác nhau](https://www.postgresql.org/docs/11/sql-createdatabase.html) có thể thay đổi được. Bạn không cần phải thay đổi bất cứ điều gì bây giờ.
 {{% /notice %}}
 6. Nhấp vào tab **SQL**
   ![SQL](/images/2/2-2/6.png)

 Tab SQL hiển thị cho bạn bản xem trước của lệnh SQL được tạo mà pgAdmin sẽ chạy.

 7. Click **Save** 
 8. Tìm cơ sở dữ liệu mới của bạn trong trình điều hướng và mở rộng nó.
    ![SQL](/images/2/2-2/7.png)

#### Tìm hiểu về Schemas
Cơ sở dữ liệu chứa một hoặc nhiều **Schemas** được đặt tên, lần lượt chứa các bảng và các đối tượng khác như dạng views và functions. Trong PostgreSQL, các đối tượng trong một Schemas có thể có các chủ sở hữu khác nhau và quyền sở hữu Schemas đó không phụ thuộc vào tên lược đồ.

Như được mô tả trong [ tài liệu về PostgreSQL ](https://www.postgresql.org/docs/11/ddl-schemas.html)


> "Tên đối tượng giống nhau có thể được sử dụng trong các Schemas khác nhau mà không có xung đột; ví dụ: cả `schema1` và `myschema` đều có thể chứa các bảng có tên mytable. Không giống như Databases, các Schemas không được phân tách một cách cứng nhắc: người dùng có thể truy cập các đối tượng trong bất kỳ Schemas nào trong cơ sở dữ liệu mà anh ta kết nối tới, nếu anh ta có đặc quyền làm như vậy.
>
> Có một số lý do tại sao người ta  muốn sử dụng schemas:
>
> - Cho phép nhiều người sử dụng cùng sử dụng một cơ sở dữ liệu mà không can thiệp lẫn nhau.
> - Để tổ chức các đối tượng cơ sở dữ liệu thành các nhóm logic để dễ quản lý hơn.
> - Các ứng dụng của bên thứ ba có thể được đưa vào các Schemas riêng biệt để chúng không thể xung đột với tên của các đối tượng khác.
>
> Các Schemas tương tự như các thư mục ở cấp độ hệ điều hành, ngoại trừ các Schemas đó không thể lồng vào nhau."

1. Click vào **Schemas** và xem các **public schema** mặc định.

   ![Schema](/images/2/2-2/8.png)



2. Nhấp chuột phải vào **Schemas** và chọn **Create**, sau đó nhấp vào **Schema**.

   ![create schema](/images/2/2-2/9.png)

3. Đặt tên cho schema là `first_schema`

   ![named schema](/images/2/2-2/10.png)

{{% notice note %}}
 Các schemas có chủ sở hữu và có các quyền bảo mật cũng như các đặc quyền mặc định cho các đối tượng mới được tạo trong schema (bạn có thể nhấp vào Security tab và the Default Permissions  trong hộp thoại Tạo schema nếu muốn).
 {{% /notice %}}


4. Nhấp vào **Save** để tạo schema.

{{% notice info %}}
**lược đồ** của PostgreSQL có thể khác với cách các cơ sở dữ liệu khác như Oracle triển khai các lược đồ. Trong Oracle, các lược đồ được ánh xạ trực tiếp 1:1 tới người dùng. Trong PostgreSQL, các lược đồ không được kết hợp trực tiếp với một người dùng (vai trò) cụ thể.
{{% /notice %}}

Như đã thảo luận trong [tài liệu](https://www.postgresql.org/docs/11/ddl-schemas.html#DDL-SCHEMAS-PORTABILITY) ,


> "Trong tiêu chuẩn SQL, khái niệm các đối tượng trong cùng một schema được sở hữu bởi những người dùng khác nhau không tồn tại. Hơn nữa, một số cách triển khai không cho phép bạn tạo các schema có tên khác với tên chủ sở hữu của chúng. Trên thực tế, các khái niệm về schema và người dùng gần như tương đương trong một hệ thống cơ sở dữ liệu chỉ triển khai hỗ trợ schema cơ bản được quy định trong tiêu chuẩn. Do đó, nhiều người dùng coi những tên đủ điều kiện thực sự bao gồm tên người dùng.tablename. Đây là cách PostgreSQL hoạt động hiệu quả nếu bạn tạo schema cho mọi người dùng. Ngoài ra, không có khái niệm về public schema trong tiêu chuẩn SQL. Để tuân thủ tối đa tiêu chuẩn, bạn không nên sử dụng (thậm chí có thể xóa) public schema."

**Lưu ý về** `search_path`
Việc tham chiếu các đối tượng thông qua các tên Đủ điều kiện, chẳng hạn như `first_schema.first_table`, rất tẻ nhạt khi viết và việc mã hóa cứng một tên lược đồ cụ thể vào một ứng dụng là không lý tưởng. Giải pháp là sử dụng các tên không đủ tiêu chuẩn, chẳng hạn như `first_table` và điều này có thể thực hiện được thông qua `search_path` của PostgreSQL.

Như đã thảo luận trong [tài liệu](https://www.postgresql.org/docs/11/ddl-schemas.html#DDL-SCHEMAS-PATH) ,

> "Hệ thống xác định bảng nào có nghĩa bằng cách đi theo **search_path**, đây là danh sách các schema cần tìm. Bảng khớp đầu tiên trong đường dẫn tìm kiếm được coi là bảng mong muốn. Nếu không có bảng khớp nào trong đường dẫn tìm kiếm, một lỗi được báo cáo, ngay cả khi tên bảng trùng khớp tồn tại trong các lược đồ khác trong cơ sở dữ liệu.
>
> Schema đầu tiên có tên trong đường dẫn tìm kiếm được gọi là schema hiện tại. Ngoài việc là lược đồ đầu tiên được tìm kiếm, nó còn là schema trong đó các bảng mới sẽ được tạo nếu lệnh `CREATE TABLE` không chỉ định tên schema."

Theo mặc định, `search_path` được đặt thành `$user,public`. Như tài liệu nêu
> "Phần tử đầu tiên chỉ định rằng một schema có cùng tên với người dùng hiện tại sẽ được tìm kiếm. Nếu không có schema như vậy tồn tại, mục nhập sẽ bị bỏ qua. Phần tử thứ hai đề cập đến lược đồ công khai mà chúng ta đã thấy."

{{% notice info %}}
Cần lưu ý rằng PostgreSQL không có khái niệm từ đồng nghĩa như một số cơ sở dữ liệu khác. Bạn có thể sử dụng `search_path` để xử lý một số, nhưng không phải tất cả, khả năng mà từ đồng nghĩa cung cấp. Để biết ví dụ về cách triển khai chức năng giống từ đồng nghĩa khác trong PostgreSQL, hãy xem [blog](https://www.dbbest.com/blog/convert-oracle-synonyms-to-postgresql/) này.
{{% /notice %}}

**Chúc mừng!**

Bạn đã tìm hiểu những kiến thức cơ bản về Databases và Schema trong PostgreSQL.