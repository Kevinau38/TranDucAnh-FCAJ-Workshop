---
title : "Chuẩn bị"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

Trong phần này bạn dựng mạng và hai EC2 instance:

| Bước | Tài nguyên tạo ra |
|---|---|
| 2.1, 2.2 | `prod-vpc` (10.10.0.0/16) và `test-vpc` (10.20.0.0/16), mỗi VPC có một private subnet trong cùng một Availability Zone |
| 2.3 | Security group, và interface endpoint cho Session Manager ở cả hai VPC |
| 2.4 | IAM role `FCJ-NVMe-EC2-Role` cho các instance |
| 2.5 | Hai instance `prod-postgres` và `test-postgres` |

Hai VPC không kết nối với nhau: không có peering, không có transit gateway và không có VPN.
