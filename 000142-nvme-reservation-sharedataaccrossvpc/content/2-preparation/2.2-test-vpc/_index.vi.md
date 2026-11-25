---
title : "Tạo VPC test"
weight : 2
chapter : false
pre : " <b> 2.2 </b> "
---

1. Trong VPC console, chọn **Create VPC** lần nữa và dùng cấu hình giống bước 2.1, chỉ khác:
   - Tên: `test`.
   - **IPv4 CIDR block**: `10.20.0.0/16`.

2. Kiểm tra cấu hình trước khi tạo:
   - (1) **NAT gateways**: **None**.
   - (2) **VPC endpoints**: **S3 Gateway**.
   - (3) Preview hiện `test-subnet-private1-ap-northeast-1a`, cùng zone với subnet production.
   - (4) Chọn **Create VPC**.

   ![NAT gateway và S3 endpoint](/images/2-preparation/2.2-vpc-nat-endpoint.png)

3. Mở **Subnets**. (1) Mỗi VPC có một private subnet: `prod-subnet-private1-ap-northeast-1a` thuộc `prod-vpc` và `test-subnet-private1-ap-northeast-1a` thuộc `test-vpc`.

   ![Các private subnet](/images/2-preparation/2.2-subnets.png)
