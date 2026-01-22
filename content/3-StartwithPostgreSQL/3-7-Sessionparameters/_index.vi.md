---
title : "Session parameters"
date :  "`r Sys.Date()`" 
weight : 7
chapter : false
pre : " <b> 3.7. </b> "
---

 Phần này sẽ nói về [PostgreSQL Parameters](https://www.postgresql.org/docs/11/runtime-config.html) . Cụ thể, nó sẽ tập trung vào các thông số có thể thay đổi ở cấp độ phiên. Đối với các tham số trên toàn hệ thống, bạn sẽ tận dụng cơ chế Parameter Group của RDS/Aurora.

 1. Trong pgAdmin, nhấp vào nút rdspg-fcj-labs, sau đó nhấp vào **Dashboard**, sau đó nhấp vào **Configuration**.
 ![parametter](/images/2/2-7/49.png)

 Phần này hiển thị cho bạn cài đặt hiện tại của tất cả **PostgreSQL parameter settings**.

 2. Trong thanh tìm kiếm, gõ **``search``**
 ![parametter](/images/2/2-7/50.png)

Lọc danh sách cài đặt tham số.

 3. Nhấp vào **first_database**, sau đó chọn biểu tượng **Query Tool**. Sau đó dán và chạy SQL sau:
 ```SELECT * FROM pg_settings where context = 'user';```
 ![parametter](/images/2/2-7/51.png)

  Lệnh này hiển thị cho bạn các phiên tham số có thể được thay đổi cho một phiên cũng như cài đặt hiện tại của chúng.

 4. Dán và chạy câu lệnh SQL sau:
 ```show search_path```
 ![parametter](/images/2/2-7/52.png)
 Lệnh ``SHOW`` là một cách khác để xem giá trị hiện tại của cài đặt của bạn.

 5. Dán và chạy câu lệnh SQL sau:
 ```show all```
  ![parametter](/images/2/2-7/53.png)
 Lệnh này chỉ cho bạn một cách khác để xem giá trị hiện tại của tất cả cài đặt tham số của bạn.

 6. Dán và chạy câu lệnh SQL sau:
 ```select * from first_table;```
 ![parametter](/images/2/2-7/54.png)
  Lệnh này không thành công vì PostgreSQL không thể tìm thấy bảng có tên **first_table** trong các lược đồ ở **search_path** hiện tại.\
 7. Dán và chạy câu lệnh SQL sau:
 ```SET search_path TO first_schema, public;   ```
 ![parametter](/images/2/2-7/55.png)

 8. Dán và chạy lại câu lệnh SQL sau:
  ```select * from first_table;```
  ![parametter](/images/2/2-7/56.png)
Lệnh này hiện đã hoạt động vì chúng ta đã cập nhật cài đặt tham số **search_path**.

 {{% notice tip%}}
 Khi thực hiện các thay đổi đối với tham số hệ thống thông qua RDS/Aurora Parameter Groups, bạn nên xác nhận rằng thay đổi tham số của mình đã được áp dụng bằng cách sử dụng một trong các kỹ thuật ở trên, chẳng hạn như "SHOW ALL", để xác nhận rằng giá trị tham số dự kiến là có hiệu lực. Đôi khi mọi người có thể nhầm lẫn vì có thể thực hiện thay đổi Nhóm tham số nhưng thay đổi cài đặt tham số thực tế không xảy ra [cho đến lần khởi động lại tiếp theo](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_Troubleshooting.html#CHAP_Troubleshooting.Parameter). Xác nhận giá trị bằng cách sử dụng "SHOW ALL" hoặc một trong các phương pháp khác có thể giúp bạn tránh nhầm lẫn và mất thời gian
 {{% /notice %}} 