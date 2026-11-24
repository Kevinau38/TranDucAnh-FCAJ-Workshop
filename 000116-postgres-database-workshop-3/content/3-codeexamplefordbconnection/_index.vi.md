---
title : "Ví dụ code kết nối database"
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

Trong chương này, bạn viết hai chương trình Node.js nhỏ để kết nối tới `rdspg-dev-db`. Cả hai đều kiểm tra chứng chỉ TLS của server bằng bộ chứng chỉ RDS. Điểm khác nhau là **thông tin kết nối lấy từ đâu**:

| | [3.1 Kết nối database đơn giản](3-1-sampledbconnect/) | [3.2 Tự động lấy thông tin đăng nhập và kết nối](3-2-autofetchingcredential/) |
|---|---|---|
| Endpoint | Biến môi trường `PGHOST` | `DescribeDBInstances` (RDS API) |
| Tên người dùng và mật khẩu | Biến môi trường `PGUSER`, `PGPASSWORD` | `GetSecretValue` (Secrets Manager) |
| Code cần gì | Mật khẩu do bạn sao chép vào | Chỉ cần tên DB instance và IAM role |
| Xoay vòng mật khẩu | Chương trình lỗi cho tới khi bạn sao chép mật khẩu mới | Tự động nhận mật khẩu mới |

Chạy mọi lệnh trong terminal Session Manager của `rdspg-dev-server`, với user `ec2-user`, trong thư mục `~/rdspg-dev-app` đã tạo ở [2.1](../2-preparation/2-1-setupenv/).
