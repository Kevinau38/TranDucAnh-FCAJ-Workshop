---
title : "Giám sát và tối ưu hiệu năng"
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

#### Tổng quan

Theo dõi tình trạng và hiệu năng của cơ sở dữ liệu là công việc hằng ngày. Trong chương này, bạn:

- Đẩy log PostgreSQL lên Amazon CloudWatch Logs và tạo CloudWatch alarm gửi email khi mức sử dụng CPU cao.
- Tạo dữ liệu thử và chạy kiểm tra tải với `pgbench`, công cụ benchmark đi kèm PostgreSQL.
- Tìm điểm nghẽn bằng **CloudWatch Database Insights**.
- Tăng cấu hình DB instance, chạy lại cùng bài kiểm tra tải và so sánh kết quả.

| Công cụ | Hiển thị gì | Chi phí trong workshop |
|---|---|---|
| CloudWatch metrics | CPU, bộ nhớ, dung lượng lưu trữ, I/O, số kết nối, mỗi phút | Miễn phí |
| CloudWatch Logs | Log PostgreSQL và log nâng cấp | Vài cent cho dữ liệu log |
| CloudWatch alarm + Amazon SNS | Gửi email khi một metric vượt ngưỡng | Miễn phí trong free tier |
| Database Insights - Standard | Tải cơ sở dữ liệu theo wait event, các câu SQL nặng nhất, lịch sử 7 ngày | Miễn phí |

{{% notice info %}}
**Performance Insights** đã được thay bằng **CloudWatch Database Insights**. Chế độ **Standard** mà bạn bật khi tạo cơ sở dữ liệu giữ lịch sử 7 ngày, không tốn thêm phí. Chế độ **Advanced** bổ sung góc nhìn toàn bộ fleet và thời gian lưu dài hơn, tính phí theo vCPU.
{{% /notice %}}
