---
title : "Các bước chuẩn bị"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

#### Tổng quan

Trong chương này, bạn tạo toàn bộ tài nguyên AWS dùng cho workshop, theo thứ tự phụ thuộc giữa chúng:

| Tài nguyên | Mục đích |
| --- | --- |
| VPC `rdspg-lab-vpc` | Mạng có public subnet và private subnet ở hai Availability Zone |
| Security group `bastion-sg`, `app-sg` | Kiểm soát truy cập SSH vào bastion host và app server |
| Security group `rds-sg` | Chỉ cho phép PostgreSQL (cổng 5432) từ `bastion-sg` và `app-sg` |
| DB subnet group `rdspg-lab-subnet-group` | Chỉ định các private subnet mà RDS sử dụng |
| EC2 instance `rdspg-app-server`, `rdspg-bastion-host` | Các máy client kết nối tới cơ sở dữ liệu |
| RDS DB instance `rdspg-lab-db` | Cơ sở dữ liệu PostgreSQL |

Toàn bộ tài nguyên được tạo ở Region **Asia Pacific (Singapore) ap-southeast-1**.
