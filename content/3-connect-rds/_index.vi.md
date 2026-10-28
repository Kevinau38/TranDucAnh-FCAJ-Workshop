---
title : "Kết nối tới RDS PostgreSQL"
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

#### Tổng quan

`rdspg-lab-db` chạy trong private subnet và không có public access, nên bạn không thể kết nối trực tiếp từ internet. Trong chương này, bạn kết nối tới database theo hai cách:

| Cách | Client | Đường đi |
|---|---|---|
| pgAdmin 4 | Máy tính của bạn | SSH tunnel qua bastion host |
| `psql` | App server trong private subnet | Trực tiếp bên trong VPC |

Cả hai cách đều hoạt động nhờ `rds-sg` cho phép cổng 5432 từ `bastion-sg` và `app-sg` đã tạo ở chương 2.
