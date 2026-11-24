---
title : "Khả năng mở rộng"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

#### Tổng quan

Khi tải tăng, bạn có thể mở rộng một cơ sở dữ liệu RDS for PostgreSQL theo hai hướng:

| Hướng | Cách làm | Phù hợp với | Giới hạn |
|---|---|---|---|
| **Chiều dọc** (scale up) | Đổi DB instance class | Nghẽn CPU hoặc bộ nhớ, tải ghi nhiều | Instance class lớn nhất; gián đoạn ngắn mỗi lần đổi |
| **Chiều ngang** (scale out) | Thêm **read replica** | Tải đọc nhiều, báo cáo, phân tích | Replica chỉ đọc và đồng bộ bất đồng bộ, nên có thể chậm hơn primary |

Trong chương này, bạn tạo read replica của `rdspg-lab-db`, promote nó thành DB instance độc lập, ôn lại cách mở rộng theo chiều dọc, và làm việc với **Multi-AZ DB cluster**: tạo Multi-AZ DB cluster từ một read replica, promote nó, rồi thêm một DB instance read replica cho cluster.

{{% notice warning %}}
Multi-AZ DB cluster chạy ba DB instance thuộc class `m`, `r` hoặc `c` có ổ NVMe cục bộ, không nằm trong free tier. Hãy xóa chúng ngay sau bước 5.4 và 5.5.
{{% /notice %}}
