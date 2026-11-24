---
title : "Kiểm tra kết quả"
weight : 3
chapter : false
pre : " <b> 4.3 </b> "
---


1. Chọn tab **Actions**, rồi chọn lần chạy **Create static.yml**. Chờ đến khi **Status** là **Success** (1). Job **deploy** hiển thị địa chỉ website (2).

![Lần chạy Static HTML](/images/4-MultiplePages/4.3-run-success.png)

{{% notice note %}}
Trang tóm tắt có thể hiện cảnh báo **Node.js 20 is deprecated** và một số action bị buộc chạy trên Node.js 24. Đây là thông báo của GitHub về phiên bản action trong workflow mẫu; việc deploy vẫn thành công. Bạn có thể bỏ qua trong bài lab này.
{{% /notice %}}

2. Mở `https://<username>.github.io/workshop1/`. Website trông giống user site, nhưng được phục vụ từ repository `workshop1`.

![Project site workshop1](/images/4-MultiplePages/4.3-site.png)
