---
title : "Các bước chuẩn bị"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

Trong phần này, bạn sẽ chuẩn bị mọi thứ mà WordPress cần trước khi cài đặt:

- Một bucket S3 chứa các file cài đặt WordPress, vì máy chủ web không có kết nối internet.
- Một VPC có public subnet (chỉ dùng cho load balancer) và private subnet (dùng cho EC2 và RDS), không có NAT gateway.
- Ba security group tạo thành một chuỗi: **internet → ALB-SG → WebServer-SG → Database-SG**.
- Các interface VPC endpoint để các instance private dùng được Session Manager và Secrets Manager.
- Một IAM role cho các EC2 instance.
- Một EC2 instance private và một cơ sở dữ liệu RDS for MySQL có Multi-AZ.

{{% notice info %}}
Bài lab này dùng Region **Asia Pacific (Tokyo) ap-northeast-1**. Nếu bạn dùng Region khác, hãy thay `ap-northeast-1` trong các câu lệnh bằng mã Region của bạn.
{{% /notice %}}

Để tìm hiểu thêm kiến thức cơ bản về VPC, EC2 và RDS, bạn có thể tham khảo các bài lab:
  - [Làm việc với Amazon VPC](https://000003.awsfcaj.com/vi/)
  - [Amazon EC2](https://000004.awsfcaj.com/vi/)
  - [Amazon RDS](https://000005.awsfcaj.com/vi/)
