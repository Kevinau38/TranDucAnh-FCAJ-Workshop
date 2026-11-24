---
title : "Triển khai WordPress trên AWS Cloud"
weight : 1
chapter : false
---
# Triển khai WordPress trên AWS Cloud

### Tổng quan

Trong bài lab này, bạn sẽ triển khai WordPress trên AWS với kiến trúc hoàn toàn private. Máy chủ web (Amazon EC2) và cơ sở dữ liệu (Amazon RDS for MySQL) chạy trong **private subnet** và không có địa chỉ IP public. Thành phần duy nhất hướng ra internet là **Application Load Balancer**, người dùng truy cập vào nó thông qua **Amazon CloudFront** bằng HTTPS.

- Các EC2 instance không có kết nối internet (không có NAT gateway). Chúng tải WordPress từ một bucket Amazon S3 qua **S3 gateway endpoint**.
- Bạn quản lý instance bằng **AWS Systems Manager Session Manager** qua các **interface VPC endpoint**, vì vậy không cần mở cổng SSH và không cần bastion host.
- Mật khẩu cơ sở dữ liệu do **AWS Secrets Manager** tạo và lưu trữ. Bạn không bao giờ phải gõ mật khẩu này.
- **Auto Scaling group** giữ cho tầng web luôn sẵn sàng trên hai Availability Zone, và **Multi-AZ** giữ một bản sao dự phòng (standby) của cơ sở dữ liệu.

![Kiến trúc](/images/architecture.png)

#### Amazon CloudFront
Amazon CloudFront là dịch vụ mạng phân phối nội dung (CDN). Trong bài lab này, CloudFront cung cấp địa chỉ HTTPS cho website và là nguồn duy nhất mà load balancer chấp nhận traffic.

#### Application Load Balancer và Target Group
Application Load Balancer nhận các request HTTP và phân phối chúng đến các EC2 instance đang khoẻ mạnh được đăng ký trong target group.

#### Auto Scaling Group
Auto Scaling group là một nhóm EC2 instance được khởi tạo từ cùng một launch template. Nhóm này thay thế các instance bị lỗi và có thể tăng hoặc giảm số lượng instance theo scaling policy.

#### Amazon RDS và Multi-AZ
Amazon RDS là dịch vụ cơ sở dữ liệu quan hệ được quản lý. Với triển khai Multi-AZ, RDS giữ một bản sao standby đồng bộ của cơ sở dữ liệu ở một Availability Zone khác và tự động chuyển đổi (failover) sang bản sao đó khi có sự cố.

#### VPC endpoint
VPC endpoint cho phép tài nguyên trong private subnet truy cập các dịch vụ AWS mà không cần đi qua internet. Bài lab dùng gateway endpoint cho Amazon S3 và interface endpoint cho Systems Manager và Secrets Manager.
