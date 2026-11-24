---
title : "Sao lưu và khôi phục"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

#### Tổng quan

Bản sao lưu chỉ có giá trị khi bạn biết cách khôi phục nó. Trong chương này, bạn xem và chỉnh automated backup của `rdspg-lab-db`, chụp manual snapshot, khôi phục snapshot đó, khôi phục cơ sở dữ liệu về thời điểm ngay trước khi một bảng bị xóa, và bảo vệ cơ sở dữ liệu bằng **AWS Backup**.

| Loại sao lưu | Ai tạo | Thời gian lưu | Khôi phục về |
|---|---|---|---|
| Automated backup | RDS, mỗi ngày một lần trong backup window, cộng nhật ký giao dịch mỗi 5 phút | 0 đến 35 ngày | Bất kỳ giây nào trong thời gian lưu (point-in-time recovery) |
| Manual snapshot | Bạn, vào bất kỳ lúc nào | Đến khi bạn xóa | Thời điểm chụp snapshot |
| Recovery point của AWS Backup | AWS Backup, theo yêu cầu hoặc theo lịch của backup plan | Theo backup plan | Thời điểm sao lưu |

{{% notice info %}}
Mỗi lần khôi phục tạo ra một DB instance **mới** với endpoint mới. DB instance nguồn vẫn chạy bình thường. Để ứng dụng dùng dữ liệu đã khôi phục, bạn trỏ ứng dụng sang endpoint mới, hoặc đổi tên các DB instance.
{{% /notice %}}

{{% notice warning %}}
Mỗi DB instance được khôi phục đều tính phí theo giờ. Hãy xóa chúng ngay khi xong từng bước.
{{% /notice %}}
