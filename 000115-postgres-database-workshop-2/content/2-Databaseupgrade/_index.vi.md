---
title : "Nâng cấp cơ sở dữ liệu"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

#### Tổng quan

Amazon RDS for PostgreSQL hỗ trợ hai kiểu nâng cấp engine:

| Kiểu nâng cấp | Ví dụ | Tương thích | Cách RDS thực hiện |
|---|---|---|---|
| Phiên bản phụ (minor) | 17.9 → 17.11 | Tương thích ngược, chỉ sửa lỗi và vá bảo mật | Thủ công, hoặc tự động trong maintenance window khi bật **Auto minor version upgrade** |
| Phiên bản chính (major) | 17.11 → 18.6 | Có thể thay đổi hành vi, catalog và extension | Chỉ khi bạn yêu cầu. RDS không bao giờ tự nâng phiên bản chính |

Trong chương này, bạn nâng cấp `rdspg-lab-db` qua hai bước: từ PostgreSQL 17.9 lên 17.11, rồi từ 17.11 lên 18.6, và kiểm tra kết quả sau mỗi bước.

{{% notice note %}}
Chương này dùng môi trường từ bước **Chuẩn bị**: `rdspg-lab-db` chạy PostgreSQL **17.9** và App server `rdspg-app-server` đã cài `psql`.
{{% /notice %}}

{{% notice warning %}}
Nâng cấp phiên bản chính không thể hoàn tác. Trong môi trường thật, hãy thử nâng cấp trên một bản sao của cơ sở dữ liệu trước, ví dụ DB instance khôi phục từ snapshot, và kiểm tra ứng dụng chạy được với phiên bản mới.
{{% /notice %}}
