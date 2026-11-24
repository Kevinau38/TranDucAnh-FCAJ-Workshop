---
title : "Triển khai image của bạn"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

Bây giờ bạn thay image nginx bằng image của bạn. Lightsail tạo **version 2** của deployment và vẫn giữ version 1, nên bạn có thể quay lại (rollback).

1. Ở tab **Deployments**, chọn **Modify your deployment**. Trong container `web`, chọn **Choose stored image** rồi chọn `:fcj-container.fcj-nginx.<số>`.

![Chọn stored image](/images/4/1-choose.png)

2. Ô **Image** giờ hiển thị stored image của bạn. Giữ nguyên cổng `80` và public endpoint `web`.

![Đã chọn stored image](/images/4/2-selected.png)

3. Kéo xuống và chọn **Save and deploy**.

![Save and deploy](/images/4/3-save.png)

4. Service hiển thị **Deploying** với image mới. Phiên bản cũ vẫn phục vụ người dùng cho đến khi phiên bản mới vượt qua health check, nên không bị gián đoạn.

![Đang triển khai version 2](/images/4/4-deploying.png)

5. Sau vài phút, trạng thái là **Running** và container dùng image của bạn.

![Đang chạy](/images/4/5-running.png)

6. Ở mục **Deployment versions**, version 2 là **Active** và version 1 là **Inactive**. Để rollback, mở menu (⋮) của version 1 và triển khai lại version đó.

![Các version của deployment](/images/4/6-versions.png)

7. Mở lại public domain, hoặc chạy lệnh `curl` ở chương 2. Bạn sẽ thấy trang **Welcome to the Amazon Lightsail Container workshop**.

![Trang của bạn](/images/4/7-curl.png)
