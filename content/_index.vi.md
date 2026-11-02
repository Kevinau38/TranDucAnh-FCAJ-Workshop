---
title : "AWS RDS PostgreSQL"
weight : 1
chapter : false
---
# Bắt đầu với Amazon RDS for PostgreSQL

#### Tổng quan

Trong workshop này, bạn xây dựng hạ tầng mạng an toàn cho một Amazon RDS for PostgreSQL DB instance, kết nối tới cơ sở dữ liệu từ máy tính của bạn và từ một EC2 instance, đồng thời thực hành các kiến thức cơ bản của PostgreSQL: database, schema, bảng, role, system catalog và truy vấn.

![Workshop architecture](/images/1-introduction/architecture.png)

#### Bạn sẽ học được

- Đặt RDS DB instance trong private subnet và kiểm soát truy cập bằng security group.
- Kết nối tới cơ sở dữ liệu private bằng pgAdmin 4 qua SSH tunnel, và bằng `psql` qua TLS từ EC2 instance.
- Hiểu cách PostgreSQL tổ chức dữ liệu và phân quyền trên Amazon RDS.
