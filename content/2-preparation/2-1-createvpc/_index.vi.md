---
title : "Tạo VPC"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 2.1. </b> "
---
 
1. Truy cập giao diện [VPC console](https://console.aws.amazon.com/vpc/) và nhấn **Create VPC**.

    ![Create a VPC](/images/preparation/1/1.png)

2. Ở phần **Resources to create**, chọn **VPC and more**.
3. Đặt tên cho VPC của bạn và chọn **CIDR block**. CIDR block là dải địa chỉ IP có sẵn cho VPC của bạn. Hãy chắc chắn rằng bạn đã chọn CIDR block đủ lớn để phù hợp với nhu cầu của bạn nhưng quá lớn dẫn đến lãng phí địa chỉ IP

    ![Create a VPC](/images/preparation/1/2.png)

4. Chọn giá trị cho **Number of public subnets**, **Number of private subnets** và **NAT Gateway**.

    ![Create a VPC](/images/preparation/1/3.png)

5. Xem lại tài nguyên VPC của bạn, sau đó click **Create VPC**

    ![Create a VPC](/images/preparation/1/4.png)
