---
title : "Chia sẻ dữ liệu giữa các VPC với EBS Multi-Attach và NVMe Reservation"
weight : 1
chapter : false
---

# Chia sẻ dữ liệu giữa các VPC với EBS Multi-Attach và NVMe Reservation

#### Tổng quan

**Multi-Attach** của Amazon EBS cho phép gắn một volume Provisioned IOPS SSD (`io1` hoặc `io2`) vào nhiều EC2 instance trong cùng một Availability Zone. Các instance không cần nằm chung VPC: volume EBS được gắn qua phần cứng Nitro của máy chủ, không đi qua mạng của VPC. Nhờ vậy bạn có thể chuyển dữ liệu giữa hai môi trường tách biệt, ví dụ từ production sang test, mà không phải mở bất kỳ đường mạng nào giữa chúng.

Khi hai instance cùng ghi được vào một ổ đĩa, cần có cách quyết định **ai được phép ghi**. Volume bật Multi-Attach hỗ trợ **NVMe reservation**, một bộ lệnh khoá (fencing) ổ đĩa theo chuẩn. Một instance giữ reservation, và chính volume sẽ từ chối lệnh ghi từ các instance khác.

Trong bài lab này, bạn sẽ:

- Tạo hai VPC là `prod-vpc` và `test-vpc`, mỗi VPC có private subnet trong cùng một Availability Zone và không có đường ra Internet.
- Chạy PostgreSQL 18 trên một instance Amazon Linux 2023 trong mỗi VPC, kết nối bằng Session Manager.
- Gắn một volume `io2` bật Multi-Attach vào cả hai instance.
- Backup một database từ instance production ra volume dùng chung trong khi instance này giữ reservation **Write Exclusive**.
- Kiểm chứng instance test đọc được volume nhưng không ghi được, rồi restore database trên instance test.
- Chuyển quyền ghi sang instance test.

#### Kiến trúc

![Kiến trúc bài lab](/images/1-introduction/architecture.png)

{{% notice info %}}
Đây là phiên bản cập nhật của workshop. Các instance giờ chạy trong private subnet và được truy cập bằng Session Manager, không có public IP, không cần SSH key và không mở cổng inbound. Phần NVMe reservation được viết lại với các lệnh đã chạy thử trên volume thật. Toàn bộ ảnh chụp ở Region **Asia Pacific (Tokyo) ap-northeast-1**.
{{% /notice %}}

{{% notice warning %}}
Chi phí: hai instance `t3.micro`, sáu interface endpoint và một volume `io2` 4 GiB, 100 IOPS tốn khoảng 0,12 USD mỗi giờ ở Tokyo. Làm phần 8 ngay khi xong bài.
{{% /notice %}}
