---
title : "Giới thiệu"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

#### Service Screener hoạt động thế nào

1. Bạn chạy công cụ trong CloudShell, chỉ định các Region và dịch vụ cần kiểm tra.
2. Với mỗi dịch vụ, công cụ liệt kê tài nguyên và đọc cấu hình bằng các API chỉ đọc.
3. Công cụ đánh giá từng tài nguyên theo một bộ quy tắc (check). Ví dụ với Amazon S3, nó kiểm tra bucket có bắt buộc HTTPS, có bật versioning hay có lifecycle policy không.
4. Kết quả được ghi vào `output.zip`: báo cáo HTML, các file JSON và CSV, cùng báo cáo theo một số khung tuân thủ (framework).

#### Báo cáo gồm những gì

| Phần | Nội dung |
|---|---|
| **Dashboard** | Số finding theo mức độ (High, Medium, Low, Informational) và theo trụ cột Well-Architected |
| **Risk Summary** | Các finding xếp theo mức ảnh hưởng, theo Region và theo dịch vụ |
| **Services** | Mỗi dịch vụ một trang, liệt kê từng finding, tài nguyên bị ảnh hưởng và link tài liệu |
| **Frameworks** | Mức độ đáp ứng các framework như CIS, NIST, AWS Foundational Technical Review (FTR) và Well-Architected Framework (WAFS) |
| **Pages** | Các trang tuỳ chọn lấy dữ liệu từ Cost Optimization Hub, Trusted Advisor và finding liên dịch vụ |

#### Service Screener và Well-Architected Tool

**AWS Well-Architected Tool** là dịch vụ trên console, nơi bạn trả lời các câu hỏi về một workload để tìm rủi ro. Service Screener bổ sung cho công cụ này: nó kiểm tra cấu hình thật của tài nguyên và có thể tự điền một phần bài review Well-Architected giúp bạn ([chương 5](../5-well-architected-integration/)).

{{% notice warning %}}
Báo cáo hiển thị tên và cấu hình tài nguyên của bạn. AWS yêu cầu báo cáo **phải được mở trên máy cục bộ và không được truy cập từ internet**. Không tải báo cáo lên website công khai hay bucket S3 công khai.
{{% /notice %}}
