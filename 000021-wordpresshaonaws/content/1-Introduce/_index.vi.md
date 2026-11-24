---
title : "Giới thiệu"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

Trong bài lab này, bạn triển khai một website WordPress có thể mở rộng theo số lượng người truy cập và vẫn hoạt động khi một Availability Zone gặp sự cố. Mọi máy chủ đều là private: phần duy nhất của kiến trúc mà internet truy cập được là Amazon CloudFront và, chỉ thông qua CloudFront, Application Load Balancer.

![Kiến trúc](/images/architecture.png)

#### Những gì bạn sẽ xây dựng

| Tầng | Dịch vụ AWS | Subnet |
|---|---|---|
| Điểm truy cập | Amazon CloudFront (HTTPS) | – |
| Cân bằng tải | Application Load Balancer, target group | Public subnet (chỉ ALB) |
| Máy chủ web | Amazon EC2 trong Auto Scaling group, launch template, AMI | Private subnet |
| Cơ sở dữ liệu | Amazon RDS for MySQL 8.4, Multi-AZ, DB snapshot | Private subnet |
| Quản trị | AWS Systems Manager Session Manager, AWS Secrets Manager, Amazon S3 | Truy cập qua VPC endpoint |

#### Nguyên tắc bảo mật của bài lab

- **Không đặt máy chủ trong public subnet.** EC2 và RDS không có địa chỉ IP public. Chỉ load balancer nằm trong public subnet.
- **Không có NAT gateway và không có internet** cho máy chủ. File cài đặt lấy từ bucket S3 của bạn qua gateway endpoint.
- **Không SSH, không bastion host.** Bạn quản lý instance bằng Session Manager qua interface endpoint.
- **Không gõ mật khẩu bằng tay.** RDS tự tạo mật khẩu master và lưu trong Secrets Manager; instance đọc trực tiếp mật khẩu này.
- **Quyền tối thiểu giữa các tầng.** Mỗi security group chỉ nhận traffic từ tầng phía trước, và load balancer chỉ nhận traffic từ CloudFront.

#### WordPress
WordPress là hệ thống quản lý nội dung miễn phí, mã nguồn mở, viết bằng PHP và dùng cơ sở dữ liệu MySQL hoặc MariaDB. Đây là một trong những nền tảng phổ biến nhất để xây dựng blog, website doanh nghiệp và cửa hàng trực tuyến.

#### Thời gian và chi phí ước tính
Bài lab mất khoảng 90 phút. Phần lớn chi phí đến từ RDS Multi-AZ, các interface endpoint và load balancer, được tính phí theo giờ. Hãy xoá toàn bộ tài nguyên ở phần 7 ngay khi hoàn thành.
