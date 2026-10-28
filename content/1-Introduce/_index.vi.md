---
title : "Giới thiệu"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

#### PostgreSQL

[PostgreSQL](https://aws.amazon.com/rds/postgresql/what-is-postgresql/) là hệ quản trị cơ sở dữ liệu quan hệ - đối tượng mã nguồn mở, đã được phát triển liên tục hơn 35 năm. PostgreSQL nổi tiếng về độ tin cậy, tuân thủ chuẩn SQL và khả năng mở rộng: bạn có thể thêm kiểu dữ liệu, hàm và extension như PostGIS hay pgvector. Nhiều doanh nghiệp và startup dùng PostgreSQL làm cơ sở dữ liệu chính cho ứng dụng nghiệp vụ, web và di động.

#### Amazon RDS for PostgreSQL

[Amazon RDS for PostgreSQL](https://aws.amazon.com/rds/postgresql/) là dịch vụ cơ sở dữ liệu được quản lý (managed). AWS vận hành PostgreSQL engine giúp bạn, để bạn tập trung vào dữ liệu và ứng dụng thay vì máy chủ cơ sở dữ liệu.

| AWS quản lý | Bạn quản lý |
| --- | --- |
| Phần cứng, hệ điều hành và cài đặt PostgreSQL | Thiết kế cơ sở dữ liệu: database, schema, bảng và index |
| Vá minor version trong maintenance window | User, role và quyền bên trong PostgreSQL |
| Sao lưu tự động và khôi phục theo thời điểm (point-in-time recovery) | Truy cập mạng: VPC, subnet và security group |
| Multi-AZ failover và read replica, khi bạn bật | Chọn instance class, storage và major version |
| Giám sát với Amazon CloudWatch và CloudWatch Database Insights | Hiệu năng truy vấn và kết nối từ ứng dụng |

Amazon RDS hỗ trợ các major version 14, 15, 16, 17 và 18 của PostgreSQL cộng đồng. Mã nguồn, ứng dụng và công cụ bạn đang dùng với PostgreSQL đều chạy được với Amazon RDS mà không cần thay đổi. Vì là dịch vụ được quản lý, master user nhận role `rds_superuser` thay vì `superuser` của PostgreSQL. Bạn sẽ thấy điều này ở chương 4.

{{% notice info %}}
Xem [Release calendars for Amazon RDS for PostgreSQL](https://docs.aws.amazon.com/AmazonRDS/latest/PostgreSQLReleaseNotes/postgresql-release-calendar.html) để biết danh sách version hiện tại và ngày kết thúc hỗ trợ tiêu chuẩn (end of standard support).
{{% /notice %}}

#### Kiến trúc của workshop

![Workshop architecture](/images/1-introduction/architecture.png)

- VPC `rdspg-lab-vpc` có một public subnet và một private subnet ở mỗi Availability Zone (2 AZ).
- RDS DB instance `rdspg-lab-db` chạy trong private subnet, không cho truy cập public. RDS yêu cầu DB subnet group có subnet ở ít nhất hai Availability Zone.
- Bastion host trong public subnet là điểm vào duy nhất từ internet, chỉ nhận SSH từ địa chỉ IP của bạn.
- App server chạy trong private subnet, không có internet. App server cài package qua S3 gateway endpoint.
- Security group dùng security group khác làm nguồn (source): `rds-sg` chỉ nhận cổng 5432 từ `bastion-sg` và `app-sg`.

#### Nội dung workshop

| Chương | Nội dung |
| --- | --- |
| 2. Các bước chuẩn bị | Tạo VPC, security group, DB subnet group, EC2 instance và RDS DB instance |
| 3. Kết nối tới RDS PostgreSQL | Kết nối bằng pgAdmin 4 qua SSH tunnel, và bằng `psql` từ app server |
| 4. Làm quen với PostgreSQL | Làm việc với database, schema, bảng, role, system catalog và truy vấn |
| 5. Tài liệu tham khảo | Tài liệu và bài blog để học tiếp |
| 6. Dọn dẹp tài nguyên | Xóa toàn bộ tài nguyên để tránh phát sinh chi phí |

#### Yêu cầu

- Tài khoản AWS có quyền với Amazon VPC, Amazon EC2 và Amazon RDS.
- Máy tính Windows có PowerShell. OpenSSH Client có sẵn trên Windows 10 và Windows 11.
- Kiến thức SQL cơ bản.

{{% notice warning %}}
Các tài nguyên trong workshop bị tính phí khi đang chạy, kể cả địa chỉ IPv4 public của bastion host. Hãy làm chương 6 khi hoàn thành.
{{% /notice %}}
