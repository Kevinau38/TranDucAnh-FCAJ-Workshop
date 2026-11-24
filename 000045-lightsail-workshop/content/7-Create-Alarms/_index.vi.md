---
title : "Tạo cảnh báo"
weight : 7
chapter : false
pre : " <b> 7. </b> "
---

**Alarm** của Lightsail theo dõi một metric và cảnh báo khi metric vượt ngưỡng. Với các gói burstable, nếu CPU chạy trên **sustainable zone** trong thời gian dài, instance sẽ dùng hết CPU burst capacity và chạy chậm lại. Alarm CPU giúp bạn biết trước khi điều đó xảy ra.

1. Mở instance `fcj-wordpress-2gb`, chọn tab **Metrics**, rồi chọn metric **CPU utilization**.

2. Ở mục **Alarms for: CPU utilization**, chọn **Create alarm**.

3. Đặt rule **Greater than or equal to** **70** phần trăm, **2** lần trong **20 phút** gần nhất. Câu màu xanh bên dưới tóm tắt lại rule.

4. Ở mục **Also notify me by**, bạn có thể chọn **Email** để nhận thông báo. Thông báo email cần một email contact đã xác minh cho Region; bạn thêm nó ở trang **Alarm notifications** trong menu bên trái. Nếu không có, alarm chỉ hiển thị banner trong Lightsail console. Chọn **Save**.

![Tạo alarm CPU](/images/7/1-alarm.png)

5. Alarm được tạo và đang ở trạng thái **OK**.

![Alarm ở trạng thái OK](/images/7/2-alarm-ok.png)

{{% notice tip %}}
Mỗi metric có tối đa hai alarm. Một số alarm hữu ích cho bài lab: **CPU burst capacity (percentage)** cho instance, **CPU utilization** hoặc **Free storage space** cho database.
{{% /notice %}}
