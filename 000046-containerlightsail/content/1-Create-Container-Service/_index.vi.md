---
title : "Tạo container service"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

1. Mở [Lightsail console](https://lightsail.aws.amazon.com/), chọn **Containers** ở menu bên trái, rồi chọn **Create container service**.

2. Kiểm tra vị trí là **Singapore, all zones (ap-southeast-1)**. Nếu không, chọn **Change AWS Region**.

![Tạo container service](/images/1/1-location.png)

3. Ở mục **Choose the power**, chọn **Nano** (512 MB RAM, 0,25 vCPU, 7 USD mỗi node mỗi tháng). Giữ scale là **×1**. Chi phí của service bằng power × scale.

![Power và scale](/images/1/2-power.png)

{{% notice info %}}
Mỗi node chạy một bản sao của mọi container trong deployment. Khi scale từ 2 trở lên, Lightsail chia lưu lượng cho các node, nên ứng dụng vẫn chạy nếu một node gặp sự cố.
{{% /notice %}}

4. Chưa thiết lập deployment ở bước này; bạn sẽ làm ở chương sau. Ở mục **Identify your service**, nhập `fcj-container`. Tên này sẽ là một phần của public domain.

![Tên service](/images/1/3-name.png)

5. Kiểm tra phần **Summary** và chọn **Create container service**.

![Tạo service](/images/1/4-create.png)

6. Service ở trạng thái **Pending** trong lúc Lightsail tạo tài nguyên, và chuyển sang **Ready** sau vài phút. **Public domain** đã hiển thị sẵn.

![Service đang tạo](/images/1/5-pending.png)
