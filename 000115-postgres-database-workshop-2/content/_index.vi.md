---
title : "AWS RDS PostgreSQL"
weight : 1
chapter : false
---
# Vận hành Amazon RDS for PostgreSQL

#### Tổng quan

Trong workshop này, bạn vận hành một Amazon RDS for PostgreSQL DB instance qua toàn bộ vòng đời: nâng cấp engine, giám sát và tối ưu hiệu năng, sao lưu và khôi phục dữ liệu, mở rộng khả năng đọc và ghi, thay đổi cấu hình bằng parameter group, và kiểm tra tính sẵn sàng cao bằng một lần failover thật.

![Workshop architecture](/images/1/architecture.png)

#### Bạn sẽ học được

- Nâng cấp PostgreSQL từ phiên bản 17 lên 18 với ít rủi ro, và kiểm tra kết quả.
- Tìm câu truy vấn chậm bằng CloudWatch, Performance Insights và `pgbench`.
- Bảo vệ dữ liệu bằng automated backup, snapshot, point-in-time recovery và AWS Backup.
- Mở rộng bằng read replica, instance class lớn hơn và Multi-AZ DB cluster.
- Thay đổi cấu hình engine bằng custom parameter group.
- Giữ database luôn sẵn sàng bằng Multi-AZ, và quan sát failover từ phía ứng dụng.
