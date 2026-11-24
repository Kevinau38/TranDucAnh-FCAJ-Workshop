---
title : "Tính sẵn sàng cao"
weight : 7
chapter : false
pre : " <b> 7. </b> "
---

#### Tổng quan

Amazon RDS for PostgreSQL có ba cách giữ database luôn sẵn sàng khi một DB instance hoặc một Availability Zone gặp sự cố:

| Lựa chọn | RDS chạy gì | Failover | Đọc từ bản sao |
|---|---|---|---|
| **Multi-AZ DB instance** | 1 primary + 1 standby ở AZ khác, replication đồng bộ | Tự động, thường 60 đến 120 giây | Không |
| **Multi-AZ DB cluster** | 1 writer + 2 standby đọc được ở 3 AZ | Tự động, thường dưới 35 giây | Có (chương 5.4) |
| **Read replica** | Bản sao bất đồng bộ, cùng hoặc khác Region | Thủ công (bạn promote) | Có (chương 5.1) |

Với **Multi-AZ DB instance**, endpoint của DB instance không đổi. Khi failover, RDS trỏ tên DNS của endpoint sang standby, standby trở thành primary mới. Ứng dụng chỉ cần kết nối lại.

Trong chương này, bạn chuyển `rdspg-lab-db` thành Multi-AZ DB instance, đăng ký nhận sự kiện failover, theo dõi endpoint từ App server, và chủ động failover bằng **Reboot with failover**.

{{% notice warning %}}
Multi-AZ DB instance có chi phí khoảng gấp đôi Single-AZ. Cuối bước 7.3, bạn chuyển `rdspg-lab-db` về lại Single-AZ.
{{% /notice %}}
