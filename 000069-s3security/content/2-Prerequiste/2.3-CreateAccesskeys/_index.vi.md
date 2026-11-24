---
title : "Kiểm tra IAM role của instance"
weight : 3
chapter : false
pre : " <b> 2.3 </b> "
---


#### Kiểm tra IAM role của instance

Bài lab cũ tạo một IAM user và lưu **access key** trên instance. Access key dài hạn là nguyên nhân phổ biến gây lộ thông tin, vì vậy phiên bản này dùng **IAM role** gắn vào instance. AWS CLI trên instance tự động nhận credential tạm thời từ instance profile.

1. Mở [IAM console](https://console.aws.amazon.com/iam/home) → **Roles** và tìm `FCJ-S3Security-EC2-Role`.
2. Tại tab **Permissions** có hai policy:
   + **AmazonSSMManagedInstanceCore** – cho phép Session Manager quản lý instance.
   + **lab-bucket-access** (inline) – chỉ cho phép `s3:*` trên bucket lab và các object trong đó.

![Quyền của IAM role](/images/2-prerequisite/2.3-iam-role.png)

{{% notice info %}}
Role chỉ có quyền trên bucket lab. Mọi lệnh **Deny** bạn thêm bằng bucket policy ở các phần sau sẽ ghi đè quyền **Allow** này – đó chính là điều bạn sẽ kiểm chứng.
{{% /notice %}}
