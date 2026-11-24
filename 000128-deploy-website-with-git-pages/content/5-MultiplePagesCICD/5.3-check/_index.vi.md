---
title : "Kiểm tra kết quả"
weight : 3
chapter : false
pre : " <b> 5.3 </b> "
---


1. Chọn tab **Actions**. Lần chạy **Create hugo.yml** đã bắt đầu từ commit của bạn (1).

![Lần chạy workflow Hugo](/images/5-MultiplePagesCICD/5.3-runs.png)

2. Mở lần chạy đó. Chờ đến khi **Status** là **Success** (1). Lần này có hai job: **build** chạy Hugo, sau đó **deploy** publish kết quả (2).

![build và deploy](/images/5-MultiplePagesCICD/5.3-run-detail.png)

{{% notice note %}}
Trang tóm tắt có thể hiện cảnh báo **Node.js 20 is deprecated** và một số action bị buộc chạy trên Node.js 24. Đây là thông báo của GitHub về phiên bản action trong workflow mẫu; việc deploy vẫn thành công. Bạn có thể bỏ qua trong bài lab này.
{{% /notice %}}

3. Mở `https://<username>.github.io/workshop2/`.

![Project site workshop2](/images/5-MultiplePagesCICD/5.3-site.png)
