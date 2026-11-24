---
title : "Lọc và nhóm dữ liệu"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

#### Lọc theo dịch vụ

Bộ lọc giúp chỉ giữ lại phần chi phí bạn muốn xem. Ở đây bạn xem chi phí của Amazon EC2.

1. Trong **Report parameters** > **Filters**, mở **Service**:
    - (1) Gõ `EC2` vào ô tìm kiếm.
    - (2) Chọn **EC2 - Other** (EBS volume và snapshot, NAT gateway, Elastic IP, truyền dữ liệu của EC2) và **EC2-Instances** (giờ chạy instance). Giữ **Includes**; **Excludes** sẽ loại bỏ các giá trị đã chọn.
    - (3) Chọn **Apply**.

    ![Lọc theo dịch vụ](/images/2/2-1.png)

2. Báo cáo được cập nhật:
    - (1) **Applied filters (1)**: hai dịch vụ hiển thị dưới **Service**.
    - (2) Phần tổng quan giờ chỉ hiển thị chi phí của 2 dịch vụ này.

    ![Báo cáo đã lọc](/images/2/2-2.png)

#### Nhóm theo usage type

3. Đặt **Group by** > **Dimension** là **Usage type** (1). Chú thích (2) giờ cho biết bạn trả tiền cho thứ gì bên trong EC2, ví dụ `NatGateway-Hours`, `NatGateway-Bytes`, `EBS:VolumeUsage.gp2` hoặc `EBS:SnapshotUsage`. Tiền tố là Region (`APS1` là Singapore, `USE2` là Ohio; không có tiền tố là US East (N. Virginia)).

    ![Nhóm theo usage type](/images/2/2-3.png)

{{% notice tip %}}
Trong tài khoản này, phần lớn chi phí "EC2" là **giờ chạy NAT gateway** chứ không phải instance. Nhóm theo usage type là cách nhanh nhất để tìm những chi phí ẩn như vậy.
{{% /notice %}}

#### Nhóm theo Region

4. Chọn **Clear all** dưới **Filters**, đặt lại **Granularity** là **Monthly**, rồi đặt **Dimension** là **Region** (1). Biểu đồ (2) so sánh chi phí của từng Region theo tháng; **Global** là chi phí của các dịch vụ toàn cầu như Amazon Route 53 hay AWS WAF cho Amazon CloudFront.

    ![Nhóm theo Region](/images/2/2-4.png)

Các dimension hữu ích khác: **Linked account** (trong AWS Organizations), **Instance type**, **Purchase option** (On-Demand, Savings Plans, Spot), **Charge type** (usage, tax, credit, refund) và **Tag**.
