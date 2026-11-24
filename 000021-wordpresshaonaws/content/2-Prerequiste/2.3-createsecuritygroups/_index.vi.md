---
title : "Tạo các security group"
weight : 3
chapter : false
pre : " <b> 2.3 </b> "
---

Bạn tạo ba security group. Mỗi security group chỉ nhận traffic từ tầng phía trước nó:

| Security group | Inbound rule | Dùng cho |
|---|---|---|
| `ALB-SG` | HTTP 80 từ internet (ở phần 6 bạn sẽ giới hạn lại chỉ cho CloudFront) | Application Load Balancer |
| `WebServer-SG` | HTTP 80 từ `ALB-SG` | Các EC2 instance chạy WordPress |
| `Database-SG` | MySQL 3306 từ `WebServer-SG` | Cơ sở dữ liệu RDS |

#### Tạo ALB-SG

1. Mở [Amazon EC2 console](https://console.aws.amazon.com/ec2/) → **Security Groups** → **Create security group**.

2. Nhập các giá trị sau:
   - (1) **Security group name**: `ALB-SG`, **Description**: `Security group for the internet-facing Application Load Balancer`.
   - (2) **VPC**: `wordpress-vpc`.
   - (3) **Inbound rules**: chọn **Add rule**, **Type** `HTTP`, **Source type** `Anywhere-IPv4`, **Description** `HTTP from the internet`.

   ![ALB-SG](/images/2-prerequisite/2.3-alb-sg.png)

3. Giữ outbound rule mặc định và chọn **Create security group**.

#### Tạo WebServer-SG

1. Chọn **Create security group** lần nữa, chọn **VPC** `wordpress-vpc` và nhập:
   - (1) **Security group name**: `WebServer-SG`, **Description**: `Security group for the private WordPress web servers`.
   - (2) **Inbound rules**: **Type** `HTTP`, **Source** – gõ `ALB-SG` rồi chọn security group trong danh sách, **Description** `HTTP from the ALB only`.

   ![WebServer-SG](/images/2-prerequisite/2.3-web-sg.png)

2. Chọn **Create security group**.

#### Tạo Database-SG

1. Chọn **Create security group** lần nữa, chọn **VPC** `wordpress-vpc` và nhập:
   - (1) **Security group name**: `Database-SG`, **Description**: `Security group for the RDS MySQL database`.
   - (2) **Inbound rules**: **Type** `MYSQL/Aurora`, **Source** – chọn `WebServer-SG`, **Description** `MySQL from the web servers only`.

   ![Database-SG](/images/2-prerequisite/2.3-db-sg.png)

2. Chọn **Create security group**.

{{% notice note %}}
Cả `WebServer-SG` và `Database-SG` đều không có rule cho SSH (cổng 22). Bạn kết nối vào instance bằng Session Manager.
{{% /notice %}}
