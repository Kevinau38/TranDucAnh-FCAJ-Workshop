---
title : "Đọc báo cáo"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

#### Tải báo cáo về

1. Trong CloudShell, chọn **Actions** > **Download file**.

![Menu Actions](/images/4/1-actions.png)

2. Nhập `/tmp/service-screener-v2/output.zip` và chọn **Download**. File được lưu vào thư mục tải về của trình duyệt.

![Download file](/images/4/2-download.png)

3. Giải nén `output.zip` trên máy. Mở thư mục `aws/<account-id>/`, bên trong có:

| File | Nội dung |
|---|---|
| `index.html` | Báo cáo Cloudscape (mở file này) |
| `index-legacy.html`, `s3.html`, `ec2.html`, ... | Báo cáo kiểu cũ, mỗi dịch vụ một trang |
| `CIS.html`, `NIST.html`, `WAFS.html`, ... | Mỗi framework một trang |
| `api-full.json`, `all.csv`, `workItem.xlsx` | Các finding ở dạng máy đọc được, dùng cho script hoặc để theo dõi việc khắc phục |

4. Mở `index.html` bằng trình duyệt.

#### Dashboard

**Dashboard** hiển thị tổng số finding theo mức độ (1) và theo trụ cột Well-Architected (2). Chọn một trụ cột hoặc mức độ để xem các finding tương ứng.

![Dashboard](/images/4/3-dashboard.png)

#### Risk Summary

**Risk Summary** xếp hạng những việc nên sửa trước. **Risk Posture** (1) đếm số finding và cho biết bao nhiêu finding có sẵn lệnh AWS CLI để sửa. **Regions by Risk** (2) cho biết finding nằm ở đâu và nhắc rằng các Region không được quét thì chưa được đánh giá.

![Risk Summary](/images/4/4-risk.png)

#### Trang dịch vụ

5. Ở mục **Services**, chọn **S3**. **Stats** (1) cho biết số tài nguyên và số quy tắc đã kiểm tra, bao nhiêu finding là **Quick Win** (không gián đoạn, không tốn phí) và bao nhiêu finding có lệnh CLI để sửa.

![Trang S3](/images/4/5-service.png)

6. Mở rộng một finding, ví dụ **TlsEnforced**. Finding giải thích rủi ro, liệt kê tài nguyên bị ảnh hưởng (bucket demo của bạn) và có link tới tài liệu AWS.

![Chi tiết finding](/images/4/6-finding.png)

{{% notice warning %}}
Các lệnh CLI trong báo cáo chỉ là gợi ý chung. Hãy đọc kỹ, thử ở môi trường không phải production và sao lưu trước khi áp dụng.
{{% /notice %}}

#### Frameworks

7. Ở mục **Frameworks**, chọn **Overview**. Trang này hiển thị tỉ lệ tuân thủ của từng framework. **Not Available** nghĩa là control đó không thể kiểm tra tự động từ cấu hình của bạn.

![Tổng quan framework](/images/4/7-frameworks.png)
