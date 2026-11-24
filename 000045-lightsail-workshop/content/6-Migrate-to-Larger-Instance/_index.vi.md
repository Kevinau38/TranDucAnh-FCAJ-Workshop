---
title : "Chuyển sang instance lớn hơn"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---

Khi blog có nhiều người truy cập hơn, bạn có thể chuyển nó sang gói có nhiều bộ nhớ hơn. Instance Lightsail không đổi gói tại chỗ được: bạn tạo một instance mới lớn hơn từ snapshot rồi chuyển static IP sang đó. Vì người truy cập dùng static IP nên địa chỉ của website không đổi.

1. Trên trang instance, trỏ chuột vào **Change plan** cạnh thông số instance. Lightsail giải thích cách làm: tạo snapshot, rồi tạo instance từ snapshot đó.

![Change plan](/images/6/1-change-plan.png)

2. Ở menu bên trái, chọn **Snapshots**. Mở rộng danh sách snapshot của `fcj-wordpress`. Khi `fcj-wordpress-snap-1` đã hoàn tất, mở menu thao tác (⋮) và chọn **Create new instance**.

![Thao tác với snapshot](/images/6/2-snapshot-menu.png)

3. Trang hiển thị snapshot nguồn và cùng Availability Zone.

![Tạo từ snapshot](/images/6/3-from-snapshot.png)

4. Ở mục **Select a size**, chọn gói **12 USD** (2 GB bộ nhớ, 60 GB SSD). Bạn có thể chọn kích thước bằng hoặc lớn hơn snapshot, nhưng không được nhỏ hơn.

![Gói lớn hơn](/images/6/4-plan.png)

5. Ở mục **Identify your instance**, nhập `fcj-wordpress-2gb`. Tag `Project` = `fcj-lightsail` được sao chép từ snapshot. Chọn **Create instance**.

![Đặt tên instance mới](/images/6/5-name.png)

6. Khi instance mới ở trạng thái **Running**, chọn **Networking** ở menu bên trái và mở static IP `fcj-wordpress-ip`. Chọn **Detach** và xác nhận. Sau đó, ở mục **Attach to an instance**, chọn `fcj-wordpress-2gb` rồi chọn **Attach**.

![Gắn vào instance mới](/images/6/6-attach.png)

7. Static IP giờ đã gắn với `fcj-wordpress-2gb`. Mở `http://<static-ip>/`: blog được phục vụ từ instance mới với đầy đủ nội dung, vì nội dung nằm trong managed database.

![Static IP đã chuyển](/images/6/7-attached.png)

8. Mở tab **Networking** của `fcj-wordpress-2gb` và kiểm tra firewall rule. Nếu rule SSH lại mở cho mọi địa chỉ, hãy giới hạn như ở [chương 4](../4-secure-the-applications/).

9. Khi instance mới đã chạy ổn, xoá instance cũ `fcj-wordpress` (**Instances** > ⋮ > **Delete**) để không phải trả tiền cho nó nữa.
