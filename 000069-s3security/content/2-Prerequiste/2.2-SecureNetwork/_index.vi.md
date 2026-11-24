---
title : "Kiểm tra security group"
weight : 2
chapter : false
pre : " <b> 2.2 </b> "
---


#### Kiểm tra security group

Phiên bản cũ của bài lab mở cổng SSH (22) cho IP của bạn. Vì bạn sẽ kết nối bằng **Session Manager**, instance không cần mở bất kỳ cổng inbound nào.

1. Mở [EC2 console](https://console.aws.amazon.com/ec2/home) → **Security Groups** và tìm `SID-instance-sg`.
2. Chọn security group và xem tab **Details**: **Inbound rules count** là **0 Permission entries**. Outbound rule duy nhất cho phép instance kết nối tới các endpoint dịch vụ AWS qua HTTPS.

![Security group không có inbound rule](/images/2-prerequisite/2.2-security-group.png)

{{% notice tip %}}
Session Manager dùng kết nối outbound từ SSM Agent tới dịch vụ Systems Manager, vì vậy bạn có thể đóng toàn bộ cổng inbound mà vẫn mở được shell trên instance.
{{% /notice %}}
