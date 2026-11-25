---
title : "Tạo VPC production"
weight : 1
chapter : false
pre : " <b> 2.1 </b> "
---

1. Mở [Amazon VPC console](https://console.aws.amazon.com/vpc/), kiểm tra Region là **Asia Pacific (Tokyo)**, rồi chọn **Create VPC**.

2. Ở **VPC settings**:
   - (1) Chọn **VPC and more**.
   - (2) Giữ **Auto-generate** và nhập `prod`. Console sẽ đặt tên tài nguyên là `prod-vpc`, `prod-subnet-private1-...`, v.v.
   - (3) **IPv4 CIDR block**: `10.10.0.0/16`.

   ![VPC settings](/images/2-preparation/2.1-vpc-settings.png)

3. Cuộn xuống và đặt:
   - (1) **Number of Availability Zones**: `1`.
   - (2) Mở **Customize AZs** và chọn `ap-northeast-1a` cho **First availability zone**. Ghi lại zone ID hiện bên cạnh (ở đây là `apne1-az4`). Cả hai VPC và volume đều phải dùng zone này.
   - (3) **Number of public subnets**: `0`.
   - (4) **Number of private subnets**: `1`.
   - (5) Phần preview hiện một private subnet ở `ap-northeast-1a`.

   ![Availability Zone và subnet](/images/2-preparation/2.1-vpc-az-subnets.png)

4. Cuộn xuống cuối form:
   - **NAT gateways**: **None**.
   - **VPC endpoints**: **S3 Gateway**. Instance cài gói từ repository của Amazon Linux 2023 (lưu trên Amazon S3) qua endpoint này.
   - Giữ **Enable DNS hostnames** và **Enable DNS resolution**. Interface endpoint ở bước 2.3 cần hai tuỳ chọn này.
   - Chọn **Create VPC** và đợi workflow báo **Success**.

{{% notice note %}}
Tên Availability Zone như `ap-northeast-1a` có thể ứng với zone vật lý khác nhau ở các tài khoản AWS khác nhau. Trong cùng một tài khoản thì ánh xạ luôn cố định, nên chọn `ap-northeast-1a` cho cả hai VPC sẽ đặt hai subnet vào cùng một zone.
{{% /notice %}}
