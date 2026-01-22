---
title : "Tables và Datatypes"
date :  "`r Sys.Date()`" 
weight : 3 
chapter : false
pre : " <b> 3.3. </b> "
---

Trong module này, chúng ta sẽ tìm hiểu vè Tables và Datatypes.
  {{% notice note %}}
  Hoàn thành module Databases và Schema trước khi tìm hiểu về Tables và Datatypes ở module này.
  {{% /notice %}}

**Bảng là gì và tại sao đôi khi tôi thấy PostgreSQL gọi chúng là Quan hệ?**\
Từ **Tài liệu PostgreSQL** ,

> "PostgreSQL là một hệ thống quản lý cơ sở dữ liệu quan hệ (RDBMS). Điều đó có nghĩa là nó là một hệ thống để quản lý dữ liệu được lưu trữ trong các mối quan hệ. Mối quan hệ về cơ bản là một thuật ngữ toán học cho bảng. Khái niệm lưu trữ dữ liệu trong bảng ngày nay phổ biến đến mức nó có vẻ như vốn có hiển nhiên, nhưng có một số cách khác để tổ chức cơ sở dữ liệu. Các tập tin và thư mục trên các hệ điều hành giống Unix là một ví dụ về cơ sở dữ liệu phân cấp. Một sự phát triển hiện đại hơn là cơ sở dữ liệu hướng đối tượng.
>
> Mỗi bảng là một tập hợp các hàng được đặt tên. Mỗi hàng của một bảng nhất định có cùng một tập hợp các cột được đặt tên và mỗi cột thuộc một kiểu dữ liệu cụ thể. Trong khi các cột có thứ tự cố định trong mỗi hàng, điều quan trọng cần nhớ là SQL không đảm bảo thứ tự của các hàng trong bảng theo bất kỳ cách nào (mặc dù chúng có thể được sắp xếp rõ ràng để hiển thị).
>
> Các bảng được nhóm thành cơ sở dữ liệu và tập hợp cơ sở dữ liệu được quản lý bởi một phiên bản máy chủ PostgreSQL duy nhất tạo thành một cụm cơ sở dữ liệu."
 
 #### Tìm hiểu về Tables

 1. Trong pgAdmin tool, hãy mở rộng **first_schema** bên trong **first_database** của bạn.
 ![table](/images/2/2-3/11.png)
 {{% notice note %}}
 Các loại đối tượng khác nhau (tables, views, functions,...) có thể là một phần của PostgreSQL schema.
 {{% /notice %}}


 2. Nhấp chuột phải vào **Tables** và chọn **Create** rồi chọn **Table**
 ![create table](/images/2/2-3/12.png)

 3. Đặt tên cho bảng của bạn **`first_table`**.
  ![named table](/images/2/2-3/13.png)
 {{% notice note %}}
 Có một tùy chọn để phân vùng bảng của bạn. Đối với **first_table** này, chúng tôi sẽ không sử dụng phân vùng bảng.
 {{% /notice %}}
 {{% notice info %}}
 Table Partitioning trong PostgreSQL là một chủ đề thú vị. Điều thú vị là việc phân vùng bảng đã phát triển qua các phiên bản PostgreSQL. Trong các phiên bản cũ hơn (phiên bản 9 trở về trước), việc phân vùng bảng được hỗ trợ gián tiếp thông qua tính năng kế thừa bảng PostgreSQL (xem bên dưới). Bắt đầu với PostgreSQL phiên bản 10, phân vùng bảng đã trở thành một tính năng khai báo gốc. Điều quan trọng cần nhớ là tập hợp chính xác các tính năng phân vùng bảng được hỗ trợ được gắn với phiên bản PostgreSQL. Vì vậy, hãy đảm bảo bạn tham khảo [tài liệu phân vùng bảng](https://www.postgresql.org/docs/11/ddl-partitioning.html) gắn liền với phiên bản PostgreSQL bạn đang sử dụng.
 {{% /notice %}}

 4. Bấm vào Columns để chuyển tới Columns tab. Sau đó bấm vào dấu **+**. Sau đó nhập `col1` cho Tên. Điền `text` cho kiểu Dữ liệu. Bấm Yes cho Not NULL. Bấm Yes cho Primary Key.
   ![ẹnded table](/images/2/2-3/14.png)
{{% notice note %}}
Có một tùy chọn để kế thừa các cột từ các bảng khác. Chúng ta sẽ không sử dụng tính kế thừa bảng cho bảng first_table này, nhưng đây là một tính năng thú vị và hữu ích cần lưu ý của PostgreSQL.
 {{% /notice %}}
 {{% notice info %}}
 Kế thừa bảng trong PostgreSQL có thể là một công cụ hữu ích cho các nhà thiết kế cơ sở dữ liệu. Kế thừa bảng cho phép bảng con kế thừa tất cả các cột của (các) bảng cha của nó. Hơn nữa, bạn cũng có thể viết truy vấn đối với các bảng cha tham chiếu đến các hàng của bảng cha và tất cả các bảng con của nó. Hãy xem ví dụ trong [tài liệu kế thừa bảng](https://www.postgresql.org/docs/11/ddl-inherit.html) để hiểu rõ hơn về các chức năng.
 {{% /notice %}}

 5. Duyệt qua các tùy chọn có sẵn trên các tab khác và kết thúc trên tab SQL. Sau đó nhấp vào Lưu.
  ![saved table](/images/2/2-3/15.png)

#### Tìm hiểu về Data types

  Như đã thảo luận trong [tài liệu](https://www.postgresql.org/docs/11/ddl-basics.html),

  "PostgreSQL bao gồm một tập hợp lớn các kiểu dữ liệu tích hợp phù hợp với nhiều ứng dụng. Người dùng cũng có thể xác định loại dữ liệu của riêng mình. Hầu hết các kiểu dữ liệu có sẵn đều có tên và ngữ nghĩa rõ ràng. Một số kiểu dữ liệu được sử dụng thường xuyên là ``integer`` cho số nguyên, ``numeric`` cho số có thể là phân số, ` ``text`` cho chuỗi ký tự, ``date`` cho ngày, ``time`` cho giá trị thời gian trong ngày và ``timestamp`` cho các giá trị chứa cả ngày và giờ."

  {{%expand "1. Numbers" %}}
  Bạn có thể đọc thêm về các loại dữ liệu số PostgreSQL [tại đây](https://www.postgresql.org/docs/11/datatype-numeric.html) . Dưới đây là một vài trích dẫn từ tài liệu:

  Loại ``integer`` là lựa chọn phổ biến [cho số nguyên] vì nó mang lại sự cân bằng tốt nhất giữa phạm vi, kích thước lưu trữ và hiệu suất.
  
  Kiểu ``numeric`` có thể lưu trữ các số có số lượng chữ số rất lớn. Nó đặc biệt được khuyến khích để lưu trữ số lượng tiền tệ và số lượng khác khi cần độ chính xác. Các phép tính với giá trị số mang lại kết quả chính xác nếu có thể, ví dụ: cộng, trừ, nhân. Tuy nhiên, việc tính toán trên các giá trị số rất chậm so với các kiểu số nguyên hoặc các kiểu thập phân [``real`` và ``double precision``].

  Các kiểu dữ liệu ``smallserial``, ``serial`` và ``bigserial`` không phải là loại đúng mà chỉ đơn thuần là sự tiện lợi về mặt ký hiệu để tạo các cột định danh duy nhất (tương tự như thuộc tính AUTO_INCREMENT được một số cơ sở dữ liệu khác hỗ trợ).
  {{% /expand%}}


  {{%expand "2. Character" %}}
  Bạn có thể đọc thêm về các kiểu dữ liệu ký tự PostgreSQL [tại đây](https://www.postgresql.org/docs/11/datatype-character.html). Dưới đây là một vài trích dẫn từ tài liệu:

  SQL định nghĩa hai loại ký tự chính: ``character varying(n)`` và ``character(n)``, trong đó n là số nguyên dương. Cả hai loại này đều có thể lưu trữ các chuỗi có độ dài tối đa n ký tự (không phải byte). ... Nếu chuỗi được lưu trữ ngắn hơn độ dài đã khai báo, các giá trị của loại ``character(n)`` sẽ được đệm khoảng trắng.

  Các ký hiệu ``varchar(n)`` và ``char(n)`` lần lượt là các bí danh cho `` ký tự thay đổi (n)`` và `` ký tự (n)``.

  Ngoài ra, PostgreSQL còn cung cấp loại ``text``, lưu trữ các chuỗi có độ dài bất kỳ.

  Không có sự khác biệt về hiệu suất giữa ba loại này, ngoài việc tăng dung lượng lưu trữ khi sử dụng loại có đệm trống và một vài chu kỳ CPU bổ sung để kiểm tra độ dài khi lưu trữ vào cột có giới hạn độ dài. Trong khi ``character(n)`` có lợi thế về hiệu năng trong một số hệ thống cơ sở dữ liệu khác, thì PostgreSQL lại không có lợi thế đó; trên thực tế, ``character(n)`` thường chậm nhất trong ba loại vì chi phí lưu trữ bổ sung. Trong hầu hết các trường hợp, nên sử dụng ``text`` hoặc ``character varying``.
  {{% /expand%}}

  {{%expand "3. Dates và Times" %}}
  Bạn có thể đọc thêm về các loại dữ liệu Dates và Times của PostgreSQL [tại đây](https://www.postgresql.org/docs/11/datatype-datetime.html).

  PostgreSQL hỗ trợ các kiểu dữ liệu ``date``, ``time`` và ``timestamp``. Kiểu dữ liệu ``time`` và ``timestamp`` có thể có hoặc không có múi giờ. Mặc định là không có múi giờ. Kiểu dữ liệu ``timestampz`` là bí danh của ``timestamp with time zone``. Các kiểu dữ liệu ``time`` và ``timestamp`` cũng có thể được chỉ định với độ phân giải chính xác 0-6 chữ số trong vài giây. Độ chính xác tối đa là 1 micro giây.

  {{% notice note %}}
  Cần lưu ý rằng kiểu dữ liệu ngày của Oracle khác với kiểu dữ liệu ngày của PostgreSQL. Một ngày của Oracle gần giống nhất với kiểu dữ liệu ``timestamp(0)`` của PostgreSQL.
  {{% /notice %}}
  {{% /expand%}}

  {{%expand "4. Other datatypes" %}}
  Bạn có thể đọc thêm về tất cả các loại dữ liệu PostgreSQL [tại đây](https://www.postgresql.org/docs/11/datatype.html) .

  PostgreSQL có một bộ kiểu dữ liệu rất phong phú và có thể mở rộng. Ví dụ: PostgreSQL có một bộ [kiểu dữ liệu dành riêng cho JSON](https://www.postgresql.org/docs/11/datatype-json.html) ``json`` và ``jsonb`` hữu ích.

  {{% /expand%}}

  **Chúc mừng!**

Bạn đã tìm hiểu những kiến thức cơ bản về PostgreSQL Tables và Datatypes..