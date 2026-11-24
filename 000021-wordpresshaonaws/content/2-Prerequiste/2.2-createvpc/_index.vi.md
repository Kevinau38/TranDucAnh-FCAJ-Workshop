---
title : "Tạo VPC"
weight : 2
chapter : false
pre : " <b> 2.2 </b> "
---

Bạn tạo một VPC trên hai Availability Zone. Mỗi zone có một public subnet, chỉ dùng cho Application Load Balancer, và một private subnet, dùng cho các EC2 instance và cơ sở dữ liệu RDS. **Không có NAT gateway**: các private subnet không có route ra internet.

1. Mở [Amazon VPC console](https://console.aws.amazon.com/vpc/) và chọn **Create VPC**.

2. Trong phần **VPC settings**:
   - (1) Chọn **VPC and more**.
   - (2) Giữ **Auto-generate** và nhập `wordpress`. Console sẽ đặt tên cho mọi tài nguyên với tiền tố này, ví dụ `wordpress-vpc`.
   - Giữ IPv4 CIDR block là `10.0.0.0/16`.

   ![Cấu hình VPC](/images/2-prerequisite/2.2-vpc-settings.png)

3. Kéo xuống và cấu hình như sau:
   - Giữ **Number of Availability Zones** = `2` và **Number of public subnets** = `2`.
   - (1) **Number of private subnets**: `2`.
   - (2) **NAT gateways**: **None**.
   - (3) **VPC endpoints**: **S3 Gateway**. Các private subnet truy cập Amazon S3 qua endpoint này.
   - Giữ chọn **Enable DNS hostnames** và **Enable DNS resolution**. Các interface endpoint ở bước 2.4 cần hai tuỳ chọn này.
   - (4) Chọn **Create VPC**.

   ![NAT gateway và S3 endpoint](/images/2-prerequisite/2.2-vpc-nat-endpoint.png)

4. Chờ đến khi workflow hiện (1) **Success**. Phần chi tiết cho thấy (2) S3 endpoint đã được tạo và (3) đã gắn vào các route table của private subnet.

   ![VPC đã được tạo](/images/2-prerequisite/2.2-vpc-success.png)

{{% notice tip %}}
Gateway endpoint cho Amazon S3 không tính phí theo giờ. Kho package của Amazon Linux 2023 cũng nằm trên Amazon S3, vì vậy các instance private có thể cài package bằng `dnf` qua endpoint này.
{{% /notice %}}
