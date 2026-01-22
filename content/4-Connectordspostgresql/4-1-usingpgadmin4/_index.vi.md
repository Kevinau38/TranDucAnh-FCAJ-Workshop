---
title : "Sử dụng pgAdmin4 để kết nối đến RDS PostgreSQL"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 4.1. </b> "
---


 {{% notice note%}}
 Trong phần này, chúng tôi sẽ sử dụng **pgAdmin4** để kết nối với RDS cho phiên bản DB PostgreSQL, vì vậy chúng tôi cần tạo DB ở chế độ công khai.\
  Bạn có thể sử dụng công cụ nguồn mở **pgAdmin4** để kết nối với phiên bản RDS cho PostgreSQL DB của mình. Bạn có thể tải xuống và cài đặt pgAdmin từ http://www.pgadmin.org/ mà không cần có phiên bản cục bộ của PostgreSQL trên máy khách của bạn
 {{% /notice %}}


 1. Khởi chạy ứng dụng **pgAdmin4** trên máy của bạn.\
 2. Trên tab **Dashboard**, chọn **Add New Server.**
     ![connecting](/images/4/4-1/14.png)
 3. Trong hộp thoại **Create - Server**, nhập tên trên tab **General** để xác định máy chủ trong pgAdmin4.

     ![connecting](/images/4/4-1/1.png)
 
 4. Trong tab **Connection**, nhập thông tin sau từ DB instance của bạn:
    - Đối với **Host**, Endpoint của RDS PostgreSQL DB, ví dụ mypostgresql.c6c8dntfzzhgv0.us-east-2.rds.amazonaws.com.
    - Đối với **Port**, nhập port được chỉ định.
    - Đối với **Username**, nhập tên người dùng mà bạn đã nhập khi tạo DB instance.
    - Đối với **Password**, nhập mật khẩu bạn đã nhập khi tạo DB instance.
    ![connecting](/images/4/4-1/2.png)
 
 5. Trong tab **Đường hầm SSH**, bật tính năng đường hầm SSH và nhập thông tin sau
    - Đối với **Host**: Tên máy chủ hoặc địa chỉ IP của máy chủ pháo đài.
    - Đối với **Port**: Số cổng SSH.
    - Đối với **Tên người dùng**: Tên người dùng mà bạn sử dụng để SSH vào máy chủ pháo đài.
    - Đối với **Identity file**: chọn cặp khóa bạn đã chọn khi tạo máy chủ pháo đài ec2.
     ![connecting](/images/4/4-1/3.png)
  
 6. Chọn **Save**.
 
    Server đã được thêm vào pgAdmin4 và bảng điều khiển của nó hiển thị trên màn hình:
    ![connecting](/images/4/4-1/6.png)

 {{% notice tip%}}
 Nếu bạn gặp bất kỳ sự cố nào khi kết nối, hãy xem [Khắc phục sự cố kết nối với RDS for PostgreSQL instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ConnectToPostgreSQLInstance.html#USER_ConnectToPostgreSQLInstance.Troubleshooting).
 {{% /notice %}}