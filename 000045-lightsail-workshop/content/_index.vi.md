---
title : "Workshop Amazon Lightsail"
weight : 1
chapter : false
---

# Workshop Amazon Lightsail

#### Tổng quan

**Amazon Lightsail** là cách đơn giản nhất để chạy một website hoặc một ứng dụng nhỏ trên AWS. Mỗi gói (plan) Lightsail gói sẵn máy chủ ảo, ổ SSD, lưu lượng truyền dữ liệu, địa chỉ IPv4 công khai và quản lý DNS với một mức giá cố định theo tháng, nên bạn không cần thiết kế VPC, security group hay EBS volume trước khi bắt đầu.

Trong workshop này, bạn dựng hai ứng dụng trên Lightsail ở Region **Singapore (ap-southeast-1)** rồi vận hành chúng:

| Ứng dụng | Cách triển khai |
|---|---|
| Blog **WordPress** | **OS blueprint** Ubuntu 24.04, một launch script và **cơ sở dữ liệu MySQL được quản lý** của Lightsail |
| Cửa hàng trực tuyến **PrestaShop** | **Application blueprint** dựng sẵn |

Khi cả hai ứng dụng đã chạy, bạn bảo mật chúng bằng firewall rule, sao lưu bằng snapshot, chuyển WordPress sang gói lớn hơn và tạo cảnh báo CPU.

![Các instance Lightsail](/images/3/4-instances.png)

#### Kiến trúc

Sơ đồ dưới đây là những gì bạn sẽ xây dựng. Cả hai ứng dụng chạy trên Lightsail instance, phía sau static IP. WordPress lưu dữ liệu trong Lightsail managed MySQL database, nên ở chương 6 bạn có thể chuyển blog sang một instance lớn hơn tạo từ snapshot mà không mất nội dung.

![Kiến trúc bài lab](/images/architecture.png)

#### Bạn cần chuẩn bị

- Một tài khoản AWS và user hoặc role có quyền dùng **Amazon Lightsail** và **AWS CloudShell**.
- Khoảng 90 phút.

{{% notice warning %}}
Tài nguyên Lightsail được tính phí theo giờ ngay từ lúc tạo, kể cả khi instance đang dừng. Bài lab dùng một database giá 15 USD/tháng và các instance giá 7 USD và 12 USD/tháng. Nếu hoàn thành trong 2–3 giờ, chi phí thấp hơn nhiều so với 1 USD. Hãy làm chương [Dọn dẹp tài nguyên](8-clean-up/) ngay khi xong.
{{% /notice %}}
