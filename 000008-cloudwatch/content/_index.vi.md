---
title : "Thực hành Amazon CloudWatch"
weight : 1
chapter : false
---

# Thực hành Amazon CloudWatch

#### Tổng quan

**Amazon CloudWatch** thu thập metric và log từ tài nguyên AWS và ứng dụng của bạn, cho phép truy vấn, vẽ biểu đồ và phát cảnh báo khi có sự cố. Trong workshop này, bạn triển khai ba EC2 instance chạy một workload mẫu cùng **CloudWatch agent**, sau đó dùng CloudWatch để:

- xem **metric**, kết hợp chúng bằng **search expression**, **math expression** và **dynamic label**;
- đọc **log**, truy vấn bằng **CloudWatch Logs Insights** và chuyển các dòng lỗi thành metric bằng **metric filter**;
- tạo **alarm** gửi email qua **Amazon SNS** khi số lỗi tăng đột biến;
- đưa mọi thứ lên một **dashboard**.

#### Kiến trúc

![Kiến trúc bài lab](/images/architecture.png)

{{% notice info %}}
Đây là phiên bản cập nhật của workshop. CloudFormation template được viết mới: chỉ tạo một public subnet, không dùng NAT gateway, dùng **Amazon Linux 2023** (arm64, t4g.micro), gắn cho instance một IAM role tối thiểu quyền (**AmazonSSMManagedInstanceCore** và **CloudWatchAgentServerPolicy**) thay cho AdministratorAccess, và cài ứng dụng mẫu ngay trong template. Toàn bộ ảnh chụp được thực hiện tại **Asia Pacific (Tokyo) ap-northeast-1**; bạn có thể dùng Region khác, miễn là làm mọi bước trong cùng một Region.
{{% /notice %}}

{{% notice warning %}}
Chi phí: ba instance t4g.micro, một vài custom metric, một alarm và một dashboard tốn chưa tới 1 USD cho vài giờ thực hành. Hãy làm phần 7 ngay khi hoàn thành.
{{% /notice %}}
