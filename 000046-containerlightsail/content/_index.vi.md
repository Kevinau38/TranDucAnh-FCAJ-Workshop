---
title : "Workshop Amazon Lightsail Containers"
weight : 1
chapter : false
---

# Workshop Amazon Lightsail Containers

#### Tổng quan

**Lightsail container service** chạy container Docker giúp bạn với giá cố định theo tháng. Bạn chọn kích thước (*power*) và số node (*scale*), đưa cho Lightsail một hoặc nhiều container image, Lightsail sẽ chạy chúng phía sau load balancer với một public domain có HTTPS. Bạn không phải quản lý server, cluster hay load balancer.

Trong workshop này, tại Region **Singapore (ap-southeast-1)**, bạn sẽ:

1. Tạo một container service.
2. Triển khai image **nginx** chính thức từ một public registry.
3. Build image của riêng bạn trong **AWS CloudShell** và push lên container service.
4. Triển khai image của bạn thành một phiên bản mới.
5. Dọn dẹp tài nguyên.

![Container service](/images/2/4-deploying.png)

#### Kiến trúc

Sơ đồ dưới đây là luồng của bài lab. Người truy cập vào container qua public domain HTTPS của container service. Version 1 của deployment lấy image nginx từ Docker Hub. Sau đó bạn build image riêng trong CloudShell, push lên container service và deploy thành version 2.

![Kiến trúc bài lab](/images/architecture.png)

#### Bạn cần chuẩn bị

- Một tài khoản AWS và user hoặc role có quyền dùng **Amazon Lightsail** và **AWS CloudShell**.
- Khoảng 60 phút.

Bạn **không** cần máy build riêng, IAM user hay access key: CloudShell đã có sẵn Docker, AWS CLI và quyền của phiên đăng nhập console.

{{% notice warning %}}
Container service bị tính phí từ lúc tạo đến lúc xoá, kể cả khi không có container nào chạy. Bài lab dùng power **Nano** (7 USD mỗi tháng, khoảng 0,01 USD mỗi giờ). Hãy làm chương [Dọn dẹp tài nguyên](5-clean-up/) ngay khi xong.
{{% /notice %}}
