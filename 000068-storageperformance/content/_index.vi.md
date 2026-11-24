---
title : "Thực hành hiệu năng lưu trữ"
weight : 1
chapter : false
---


# Thực hành hiệu năng lưu trữ

#### Tổng quan

Amazon S3 và Amazon EFS được thiết kế để mở rộng theo chiều ngang: dữ liệu được phân tán trên rất nhiều máy chủ, nên một client đạt hiệu năng tốt nhất khi gửi **nhiều request cùng lúc**. Trong bài lab này, bạn chạy cùng một tác vụ truyền dữ liệu theo nhiều cách khác nhau từ một EC2 instance và đo sự chênh lệch: một luồng so với nhiều luồng, một object lớn so với nhiều phần, một thư mục so với nhiều thư mục.

| Phần | Bạn đo những gì |
|---|---|
| Amazon S3 | Tốc độ upload với 1 đến 20 luồng, lệnh `aws s3 sync`, nhiều object nhỏ và thao tác copy phía server |
| Amazon EFS | Tạo nhiều file nhỏ, kích thước I/O và tần suất sync, ghi đa luồng, và các công cụ copy phổ biến (`rsync`, `cp`, `fpsync`, GNU `parallel`) |

#### Kiến trúc

![Kiến trúc bài lab](/images/architecture.png)

Một CloudFormation template tạo toàn bộ môi trường lab: một VPC có một public subnet, một EC2 instance (Amazon Linux 2023, Graviton) gắn ổ EBS có sẵn 2.000 file mẫu, một EFS file system mount tại `/efs` và một S3 bucket truy cập qua gateway endpoint. Bạn mở shell trên instance bằng **Session Manager**, nên instance không mở port inbound nào và không cần SSH key.

{{% notice info %}}
Đây là phiên bản cập nhật của workshop. Template mới thay cho template của Storage Immersion Day: không còn tạo IAM user hay access key, và instance dùng IAM role chỉ được truy cập bucket của bài lab. Toàn bộ ảnh chụp được thực hiện tại **Asia Pacific (Tokyo) ap-northeast-1**. Bạn có thể dùng Region khác, miễn là làm mọi bước trong cùng một Region.
{{% /notice %}}

{{% notice warning %}}
Chi phí: instance c7g.large, các ổ EBS, EFS và các object thử nghiệm trên S3 tốn chưa tới 1 USD nếu bạn hoàn thành bài lab trong khoảng một giờ. Hãy làm phần 5 ngay khi xong.
{{% /notice %}}

Thời gian trong ảnh chụp là kết quả của một lần chạy. Số liệu của bạn sẽ khác, nhưng xu hướng sẽ giống nhau.
