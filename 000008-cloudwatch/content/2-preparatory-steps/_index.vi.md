---
title : "Chuẩn bị"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---


Trong phần này, bạn tạo môi trường lab bằng một CloudFormation stack.

1. Tải template [fcj-cloudwatch-workshop.yaml](/files/fcj-cloudwatch-workshop.yaml).
2. Mở console **CloudFormation**, chọn **Create stack** (1), rồi **With new resources (standard)**.

![Create stack](/images/2-preparation/2.1-create-stack.png)

3. Giữ **Choose an existing template**, chọn **Upload a template file** (1), chọn **Choose file** và chọn file template vừa tải (2). Chọn **Next** (3).

![Upload template](/images/2-preparation/2.1-upload-template.png)

4. Tại **Specify stack details**:
   - **Stack name** (1): `FCJ-CloudWatch-Workshop`.
   - **InstanceType** (2): giữ `t4g.micro`.
   - **LatestAmiId**: giữ mặc định, giá trị này tự trỏ tới AMI Amazon Linux 2023 arm64 mới nhất.
   - **PermissionsBoundaryArn** (3): để trống nếu dùng tài khoản riêng. Nếu tổ chức của bạn bắt buộc gắn permissions boundary cho IAM role mới, hãy nhập ARN của nó. Ảnh chụp dùng boundary bắt buộc của tài khoản chung FCJ.
   - Chọn **Next** (4).

![Thông tin stack](/images/2-preparation/2.1-stack-details.png)

5. Tại **Configure stack options**, giữ mặc định, cuộn xuống, chọn **I acknowledge that AWS CloudFormation might create IAM resources with custom names** (1) rồi chọn **Next** (2).

![Xác nhận tạo IAM](/images/2-preparation/2.1-capabilities.png)

6. Kiểm tra lại và chọn **Submit** (1).

![Submit](/images/2-preparation/2.1-submit.png)

7. Chờ khoảng 3 phút đến khi stack có trạng thái **CREATE_COMPLETE** (1).

![CREATE_COMPLETE](/images/2-preparation/2.1-create-complete.png)

{{% notice info %}}
Sau khi stack hoàn tất, instance cần thêm một đến hai phút để cài CloudWatch agent và bắt đầu gửi dữ liệu. Hãy chờ khoảng 10 phút trước khi sang phần 3 để biểu đồ có dữ liệu.
{{% /notice %}}

Stack tạo các tài nguyên:

| Tài nguyên | Tên |
|---|---|
| VPC, public subnet, internet gateway, route table | `FCJ-CloudWatch-VPC`, `FCJ-CloudWatch-Public-Subnet`, ... |
| Security group không có inbound rule | `FCJ-CloudWatch-Instance-SG` |
| IAM role và instance profile | `FCJ-CloudWatch-EC2-Role-<region>` |
| Log group | `/fcj/cloudwatch-workshop/var/log/messages` |
| Ba EC2 instance | `FCJ-CloudWatch-Instance-A`, `-B`, `-C` |
