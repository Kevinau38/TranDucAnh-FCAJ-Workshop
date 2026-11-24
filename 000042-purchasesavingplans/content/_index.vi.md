---
title : "Savings Plans, Reserved Instances và Reserved DB Instances"
weight : 1
chapter : false
---

# Savings Plans, Reserved Instances và Reserved DB Instances

#### Tổng quan

Giá On-Demand rất đơn giản: bạn trả theo giây hoặc theo giờ và ngừng trả khi dừng tài nguyên. Với các workload chạy liên tục, AWS đưa ra mức giá thấp hơn để đổi lấy **cam kết 1 hoặc 3 năm**. Workshop này giải thích ba mô hình cam kết cho compute và cơ sở dữ liệu, và hướng dẫn bạn đi qua các trang mua trong AWS console:

| Mô hình | Bạn cam kết điều gì | Áp dụng cho |
|---|---|---|
| **Savings Plans** | Một số tiền mỗi giờ (ví dụ 0,10 USD/giờ) | EC2, Fargate, Lambda, SageMaker AI và các cơ sở dữ liệu được quản lý, tuỳ loại plan |
| **EC2 Reserved Instances** | Một instance type, platform và Region cụ thể | Amazon EC2 |
| **Reserved DB instance** | Một DB engine, instance class và Region cụ thể | Amazon RDS và Amazon Aurora |

{{% notice warning %}}
Workshop này không bao giờ hoàn tất việc mua. Savings Plans, Reserved Instances và reserved DB instance là cam kết 1 hoặc 3 năm: khi đã gửi đơn, bạn trả tiền cho mọi giờ trong thời hạn dù có dùng hay không. Bạn chỉ đi đến giỏ hàng, rồi xoá giỏ hàng.
{{% /notice %}}

#### Bạn cần chuẩn bị

- Một tài khoản AWS và user hoặc role có quyền mở **Billing and Cost Management**, Amazon EC2 console và Amazon RDS console.
- Workshop không tạo tài nguyên nào nên **không phát sinh chi phí**.
