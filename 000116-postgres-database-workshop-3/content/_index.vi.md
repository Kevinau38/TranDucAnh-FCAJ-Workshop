---
title : "AWS RDS PostgreSQL for Developers"
weight : 1
chapter : false
---
# Amazon RDS for PostgreSQL dành cho developer

#### Tổng quan

Trong workshop này, bạn viết code Node.js kết nối tới Amazon RDS for PostgreSQL theo cách một ứng dụng production nên làm: qua TLS có kiểm tra chứng chỉ server, không có mật khẩu trong code hay file cấu hình, và dùng thông tin đăng nhập do AWS tự xoay vòng. Sau đó bạn triển khai một ứng dụng web nhỏ đọc và ghi vào database.

![Workshop architecture](/images/1/architecture.png)

#### Bạn sẽ học được

- Để RDS quản lý mật khẩu master trong AWS Secrets Manager, và đọc nó từ EC2 instance bằng IAM role.
- Kết nối từ Node.js bằng driver `pg` qua TLS với `verify-full`.
- Tìm endpoint và thông tin đăng nhập lúc chạy bằng AWS SDK for JavaScript v3.
- Xây dựng và chạy ứng dụng web CRUD với Express, Handlebars và RDS for PostgreSQL.
