---
title : "Chuẩn bị"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

#### Tổng quan

Trong chương này, bạn tạo môi trường của workshop và tìm hiểu cách RDS lưu mật khẩu database trong AWS Secrets Manager.

| Tài nguyên | Tên | Cấu hình |
|---|---|---|
| VPC | `rdspg-dev-vpc` | `10.0.0.0/16`, 1 public subnet, 2 private subnet, internet gateway |
| Security group | `dev-server-sg`, `rds-dev-sg` | Port 3000 từ IP của bạn → dev server; PostgreSQL 5432 từ dev server |
| IAM role | `FCJ-rdspg-dev-ec2-role` | Session Manager, `rds:DescribeDBInstances` trên `rdspg-dev-db`, `secretsmanager:GetSecretValue` trên các secret do RDS quản lý |
| EC2 | `rdspg-dev-server` | Amazon Linux 2023, `t3.micro`, không có key pair |
| RDS | `rdspg-dev-db` | PostgreSQL 18, `db.t4g.micro`, mật khẩu master quản lý trong Secrets Manager, database `pglab` |
