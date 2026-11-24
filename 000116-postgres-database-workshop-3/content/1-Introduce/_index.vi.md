---
title : "Giới thiệu"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

#### Tổng quan

Hai workshop trước nhìn RDS for PostgreSQL từ phía người quản trị database. Workshop này nhìn từ phía **developer**: code của bạn tìm database thế nào, xác thực ra sao, và giữ kết nối an toàn bằng cách nào.

| Chương | Bạn học cách |
|---|---|
| 2. Chuẩn bị | Tạo mạng, database có secret do RDS quản lý, và một server phát triển có IAM role |
| 3. Code mẫu | Kết nối từ Node.js bằng biến môi trường, rồi bằng AWS SDK và Secrets Manager |
| 4. Triển khai ứng dụng | Chạy ứng dụng web FCJ User Management trên server phát triển |
| 5. Dọn dẹp | Xoá toàn bộ tài nguyên của workshop |

#### Kiến trúc

![Kiến trúc](/images/1/architecture.png)

- **`rdspg-dev-server`** là EC2 instance trong public subnet. Bạn mở shell trên nó bằng **AWS Systems Manager Session Manager**, nên không cần SSH key và không mở port SSH. Security group `dev-server-sg` chỉ cho phép port 3000 (ứng dụng web) từ IP của bạn.
- Instance profile gắn IAM role **`FCJ-rdspg-dev-ec2-role`** cho server. Ứng dụng dùng thông tin đăng nhập tạm thời của role để gọi RDS API và Secrets Manager; không có access key nào trên server.
- **`rdspg-dev-db`** là RDS for PostgreSQL 18 DB instance trong hai private subnet. Security group `rds-dev-sg` chỉ cho phép port 5432 từ `dev-server-sg`.
- RDS lưu user master và mật khẩu trong một **secret của AWS Secrets Manager** (`rds!db-...`), và xoay vòng mật khẩu mỗi 7 ngày.

| Công cụ | Phiên bản trong workshop |
|---|---|
| Amazon Linux | 2023 |
| Node.js / npm | 24 / 11 (gói `nodejs24`) |
| PostgreSQL client / server | 18.6 / 18.3 |
| `pg`, `express`, `express-handlebars` | 8.x, 4.x, 8.x |
| AWS SDK for JavaScript | v3 (`@aws-sdk/client-rds`, `@aws-sdk/client-secrets-manager`) |

{{% notice note %}}
Workshop dùng Region Asia Pacific (Singapore) `ap-southeast-1` trong nội dung và script. Ảnh chụp đã che Region, account ID, địa chỉ IP public và endpoint. Bạn có thể dùng bất kỳ Region nào có RDS for PostgreSQL 18.
{{% /notice %}}

#### Chi phí

`db.t4g.micro`, `t3.micro` và 20 GiB `gp3` nằm trong AWS Free Tier. Bạn trả tiền cho secret do RDS quản lý (0,40 USD mỗi tháng, tính theo giờ) và địa chỉ IPv4 public của server (0,005 USD mỗi giờ). Hãy chạy chương 5 khi làm xong.

Tiếp tục với [2. Chuẩn bị](../2-preparation/).
