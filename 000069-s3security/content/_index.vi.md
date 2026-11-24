---
title : "Thực hành bảo mật S3"
weight : 1
chapter : false
---

# Thực hành bảo mật S3

#### Tổng quan

Trong bài lab này, bạn sẽ áp dụng các thực hành bảo mật tốt nhất để bảo vệ dữ liệu lưu trên **Amazon S3**. Bạn thao tác từ một EC2 instance được gắn IAM role, và sử dụng **bucket policy**, **ACL**, **S3 Block Public Access**, **VPC gateway endpoint**, rule của **AWS Config** và **IAM Access Analyzer** để kiểm soát và kiểm tra ai có thể truy cập bucket của bạn.

![Kiến trúc bài lab](/images/1-introduction/architecture.png)

{{% notice info %}}
Đây là phiên bản cập nhật của workshop. Môi trường lab được tạo bằng một CloudFormation template duy nhất, và EC2 instance dùng **IAM role** cùng **Session Manager** thay cho IAM user, access key và SSH. Toàn bộ ảnh chụp được thực hiện tại **Asia Pacific (Tokyo) ap-northeast-1**; bạn có thể dùng Region khác, miễn là mọi bước đều làm trong cùng một Region.
{{% /notice %}}
