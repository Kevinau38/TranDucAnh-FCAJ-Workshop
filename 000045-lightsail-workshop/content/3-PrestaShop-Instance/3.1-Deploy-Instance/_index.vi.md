---
title : "Triển khai instance"
weight : 1
chapter : false
pre : " <b> 3.1. </b> "
---

1. Trong Lightsail console, chọn **Instances**, rồi **Create instance**. Giữ vị trí **Singapore, Zone A**.

2. Ở mục **Select a platform**, chọn **Linux apps**. Ở mục **Select a blueprint**, chọn **PrestaShop**.

![Blueprint PrestaShop](/images/3/1-blueprint.png)

{{% notice info %}}
Blueprint PrestaShop có nhãn **Originally packaged by Bitnami (no longer maintained)**. Bitnami đã ngừng cập nhật các image này: sau ngày 19/05/2026 chúng không còn nhận bản cập nhật mới, và từ ngày 19/05/2027 bạn không thể tạo instance mới từ chúng. Với cửa hàng dùng lâu dài, hãy cài PrestaShop trên OS blueprint, giống cách bạn đã cài WordPress.
{{% /notice %}}

3. Ở mục **Select a size**, chọn gói **12 USD** (2 GB bộ nhớ). Với gói 1 GB, Lightsail cảnh báo PrestaShop cần gói lớn hơn. PrestaShop không hỗ trợ gói **IPv6-only**, nên giữ **Dual-stack**.

![Gói PrestaShop](/images/3/2-plan.png)

4. Ở mục **Instance name**, nhập `fcj-prestashop`. Thêm tag `Project` = `fcj-lightsail`. Chọn **Create instance**.

![Tên và tag](/images/3/3-name-tag.png)

5. Chờ 3–5 phút để instance khởi động lần đầu xong.
