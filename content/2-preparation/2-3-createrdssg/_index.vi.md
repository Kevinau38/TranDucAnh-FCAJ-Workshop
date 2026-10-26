---
title : "Tạo Security Group cho RDS"
weight : 3
chapter : false
pre : " <b> 2.3. </b> "
---

#### Tổng quan

Ở bước này, bạn tạo `rds-sg` cho Amazon RDS for PostgreSQL. Security group này chỉ cho phép kết nối PostgreSQL (cổng **5432**) từ hai nguồn:

| Nguồn | Lý do |
|---|---|
| `app-sg` | App server kết nối tới database bằng `psql` |
| `bastion-sg` | pgAdmin trên máy bạn kết nối qua SSH tunnel, nên kết nối tới RDS xuất phát từ bastion host |

{{% notice info %}}
`rds-sg` không cho phép bất kỳ địa chỉ IP nào, kể cả IP của bạn. Máy tính của bạn chỉ tới được database thông qua bastion host.
{{% /notice %}}

#### Tạo rds-sg

1. Trong [Amazon EC2 console](https://console.aws.amazon.com/ec2/), chọn **Security Groups**, sau đó chọn **Create security group**. Trong **Basic details**:
    - (1) **Security group name**: `rds-sg`
    - (2) **Description**: `Allow PostgreSQL from app server and bastion host`
    - (3) **VPC**: `rdspg-lab-vpc`

    ![rds-sg basic details](/images/2-preparation/2.3/1.png)

2. Trong **Inbound rules**, thêm hai rule:
    - (1) **Type**: `PostgreSQL`, **Source type**: `Custom`, **Source**: `app-sg`
    - (2) **Type**: `PostgreSQL`, **Source type**: `Custom`, **Source**: `bastion-sg`

    ![rds-sg inbound rules](/images/2-preparation/2.3/2.png)

{{% notice note %}}
Khi chọn type **PostgreSQL**, protocol tự động là TCP và cổng là 5432.
{{% /notice %}}

3. Giữ mặc định outbound rules và chọn **Create security group**. `rds-sg` có hai inbound rule trên cổng 5432.

    ![rds-sg created](/images/2-preparation/2.3/3.png)
