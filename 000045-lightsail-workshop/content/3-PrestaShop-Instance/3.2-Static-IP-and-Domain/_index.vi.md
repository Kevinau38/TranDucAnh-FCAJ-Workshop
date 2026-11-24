---
title : "Gắn static IP và đặt tên miền"
weight : 2
chapter : false
pre : " <b> 3.2. </b> "
---

PrestaShop cũng lưu địa chỉ của chính nó và chuyển hướng mọi người truy cập về địa chỉ đó. Sau khi gắn static IP, bạn cần báo cho PrestaShop dùng địa chỉ mới.

1. Tạo và gắn static IP tên `fcj-prestashop-ip`. Bạn có thể làm lại các bước ở [2.2](../../2-wordpress-instance/2.2-static-ip/) trên console, hoặc chạy các lệnh sau trong CloudShell:

```bash
aws lightsail allocate-static-ip --static-ip-name fcj-prestashop-ip
aws lightsail attach-static-ip --static-ip-name fcj-prestashop-ip --instance-name fcj-prestashop
```

2. Trang **Instances** giờ hiển thị biểu tượng ghim cạnh IP công khai của cả hai instance: chúng đang dùng static IP.

![Các instance dùng static IP](/images/3/4-instances.png)

3. Trong CloudShell, đặt tên miền của PrestaShop thành static IP bằng công cụ `configure_app_domain` của Bitnami, rồi kiểm tra chuyển hướng. Dòng cuối phải là `302 http://<static-ip>/`.

```bash
PSIP=$(aws lightsail get-static-ip --static-ip-name fcj-prestashop-ip --query staticIp.ipAddress --output text)
./lsssh.sh fcj-prestashop "sudo /opt/bitnami/configure_app_domain --domain $PSIP"
./lsssh.sh fcj-prestashop "curl -s -o /dev/null -w '%{http_code} %{redirect_url}\n' http://localhost/"
```

{{% notice note %}}
`configure_app_domain` có thể in ra `Job for bitnami-config-hostname.service failed`. Tên miền vẫn được cập nhật; kết quả chuyển hướng `302` về static IP xác nhận điều đó.
{{% /notice %}}

4. Mở `http://<static-ip>/` trên trình duyệt. Cửa hàng demo PrestaShop đã chạy.

![Cửa hàng PrestaShop](/images/3/5-storefront.png)

{{% notice tip %}}
User name và mật khẩu quản trị PrestaShop được lưu trên instance. Để xem, chạy `./lsssh.sh fcj-prestashop 'sudo cat /home/bitnami/bitnami_credentials'` trong CloudShell. Không chia sẻ kết quả này.
{{% /notice %}}
