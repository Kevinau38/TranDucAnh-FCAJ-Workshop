---
title : "Giới thiệu"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

#### Tổng quan

Ở workshop thứ nhất, bạn đã tạo một cơ sở dữ liệu RDS for PostgreSQL và kết nối vào nó. Workshop này đi vào **vận hành hằng ngày** (day-2 operations) của chính cơ sở dữ liệu đó: những việc giúp một cơ sở dữ liệu production luôn được cập nhật, chạy nhanh, được bảo vệ và sẵn sàng.

| Chương | Bạn học cách |
|---|---|
| 2. Nâng cấp cơ sở dữ liệu | Nâng cấp phiên bản phụ và phiên bản chính (17.9 → 17.11 → 18.6) |
| 3. Giám sát hiệu năng | Đẩy log, tạo alarm, chạy kiểm tra tải, tìm điểm nghẽn bằng Database Insights, scale up |
| 4. Sao lưu và khôi phục | Dùng automated backup, snapshot, khôi phục về một thời điểm và AWS Backup |
| 5. Khả năng mở rộng | Tạo và promote read replica, chuyển sang Multi-AZ DB cluster |
| 6. Parameter group | Đổi cấu hình PostgreSQL bằng parameter group tùy chỉnh |
| 7. Tính sẵn sàng cao | Chuyển sang Multi-AZ và thử failover |
| 8. Tìm hiểu thêm về PostgreSQL | Các tùy chọn hữu ích của `pgbench` và bước tiếp theo |
| 9. Dọn dẹp | Xóa mọi tài nguyên của workshop |

#### Kiến trúc

![Architecture](/images/1/architecture.png)

- **`rdspg-bastion-host`** ở public subnet là máy chủ duy nhất bạn truy cập từ internet, bằng SSH và chỉ từ địa chỉ IP của bạn.
- **`rdspg-app-server`** ở private subnet chạy `psql` và `pgbench`. Bạn vào nó qua bastion host bằng SSH `ProxyJump`.
- **`rdspg-lab-db`** là DB instance RDS for PostgreSQL, nằm trong các private subnet. Security group `rds-sg` của nó chỉ cho phép cổng 5432 từ App server và bastion host.
- App server cài gói từ kho Amazon Linux qua **S3 gateway endpoint**, không cần NAT gateway.
- DB subnet group có private subnet ở **ba** Availability Zone, điều mà Multi-AZ DB cluster ở chương 5 yêu cầu.

{{% notice note %}}
Phần chữ và sơ đồ của workshop dùng Region Asia Pacific (Singapore) `ap-southeast-1`. Ảnh chụp màn hình đã che Region, account ID và endpoint. Bạn có thể dùng bất kỳ Region nào có RDS for PostgreSQL 17 và 18.
{{% /notice %}}

#### Chi phí

Phần lớn tài nguyên của workshop nằm trong AWS Free Tier: `db.t4g.micro`, `t3.micro`, 20 GiB lưu trữ `gp3`, CloudWatch metrics và Database Insights Standard. Một số bước dùng tài nguyên **không** miễn phí, tính phí theo giờ, chỉ trong vài phút:

| Tài nguyên | Chương |
|---|---|
| `db.t4g.xlarge` | 3.4 |
| DB instance được khôi phục, read replica | 4.4, 4.5, 5.1 |
| Multi-AZ DB cluster (3 × `db.c6gd.medium`) | 5.4, 5.5 |
| Multi-AZ DB instance (standby) | 7 |

Hãy xóa từng tài nguyên ngay khi bước tương ứng yêu cầu, và làm chương 9 khi kết thúc.

#### Nội dung

1. [Chuẩn bị](1-1-preparation/)
