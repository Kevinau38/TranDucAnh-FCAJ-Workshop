---
title : "Gắn static IP"
weight : 2
chapter : false
pre : " <b> 2.2. </b> "
---

Địa chỉ IPv4 công khai mà Lightsail cấp cho instance sẽ thay đổi khi bạn dừng rồi khởi động lại instance. WordPress lưu URL của chính nó trong database, vì vậy bạn gắn **static IP** cho instance trước khi cài WordPress. Static IP miễn phí khi đang gắn với một instance.

1. Ở menu bên trái, chọn **Networking**, rồi **Create static IP**.

![Trang Networking](/images/2/6-networking.png)

2. Ở mục **Attach to an instance**, chọn `fcj-wordpress`. Ở mục **Identify your static IP**, nhập `fcj-wordpress-ip`. Chọn **Create**.

![Gắn static IP](/images/2/7-attach.png)

3. Static IP được tạo và gắn vào instance. Ghi lại địa chỉ này: đây là địa chỉ blog của bạn.

![Static IP](/images/2/8-static-ip.png)

{{% notice warning %}}
Static IP **không gắn** với instance nào sẽ bị tính phí theo giờ. Hãy xoá nó nếu bạn đã gỡ ra và không dùng nữa.
{{% /notice %}}
