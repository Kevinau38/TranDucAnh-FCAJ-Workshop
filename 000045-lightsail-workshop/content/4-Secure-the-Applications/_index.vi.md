---
title : "Bảo mật ứng dụng"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

Mỗi instance Lightsail có một **firewall** kiểm soát lưu lượng đi vào. Mặc định firewall cho phép HTTP (cổng 80) và SSH (cổng 22) từ mọi địa chỉ IPv4 và IPv6. Người truy cập cần HTTP, nhưng SSH mở ra toàn internet là mục tiêu quen thuộc của các bot dò mật khẩu. Bạn sẽ giới hạn SSH để chỉ **Lightsail browser SSH client** kết nối được.

{{% notice warning %}}
Sau thay đổi này, script `lsssh.sh` trong CloudShell không còn kết nối được tới instance. Hãy làm chương này sau khi đã hoàn thành chương 2 và 3. Nếu cần truy cập lại từ CloudShell, thêm địa chỉ IP nguồn của bạn vào rule SSH.
{{% /notice %}}

1. Mở instance `fcj-wordpress` và chọn tab **Networking**. Bảng **Firewall rules** hiển thị các rule mặc định.

![Firewall rule mặc định](/images/4/1-firewall.png)

2. Ở rule **SSH**, chọn biểu tượng sửa (bút chì).

3. Ở mục **Source IP addresses**, chọn **Remove** cho cả **Anywhere IPv4** và **Anywhere IPv6**. Giữ **Lightsail browser SSH client** với **Enable IPv4** và **Enable IPv6** được chọn. Chọn **Update rule**.

![Sửa rule SSH](/images/4/2-edit-rule.png)

4. Rule SSH giờ hiển thị **Lightsail browser SSH only**.

![SSH đã được giới hạn](/images/4/3-ssh-only.png)

5. Ở tab **Connect**, chọn **Connect using SSH** để kiểm tra SSH trên trình duyệt vẫn hoạt động.

![SSH trên trình duyệt](/images/4/4-browser-ssh.png)

6. Lặp lại bước 1–4 cho `fcj-prestashop`.

{{% notice tip %}}
Một số thực hành tốt khác cho instance Lightsail: cập nhật hệ điều hành thường xuyên (`sudo apt update && sudo apt upgrade`), chỉ mở những cổng cần thiết, và dùng HTTPS với chứng chỉ TLS cho mọi trang có đăng nhập.
{{% /notice %}}
