---
title : "Đề xuất Savings Plans"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

#### Tổng quan

AWS xem phần sử dụng On-Demand của bạn trong 7, 30 hoặc 60 ngày qua và đề xuất mức cam kết theo giờ giúp tiết kiệm nhiều nhất. Phần đề xuất này miễn phí và được cập nhật ít nhất mỗi ngày một lần.

1. Trong menu **Savings Plans**, chọn **Recommendations**:
    - (1) **Savings Plans type**: Compute, EC2 Instance, SageMaker hoặc Database.
    - (2) **Savings Plans term**: 1 hoặc 3 năm.
    - (3) **Payment option**: All, Partial hoặc No upfront.
    - (4) **Based on the past**: khoảng thời gian sử dụng dùng để tính toán.

    ![Tham số đề xuất](/images/2/2-1.png)

2. Cuộn xuống phần kết quả:
    - (1) **Current monthly on-demand spend**, **Estimated monthly spend** và **Estimated monthly savings** so sánh hoá đơn trước và sau khi mua theo đề xuất.
    - (2) Trong bảng **Recommended Savings Plans**, bạn có thể chọn một đề xuất và chọn **Add Savings Plans to cart**.
    - (3) Trong tài khoản lab, bảng thường trống: AWS không đưa ra đề xuất khi không có cơ hội tiết kiệm, hoặc khi chi phí On-Demand trung bình của bạn dưới **0,10 USD mỗi giờ**.

    ![Đề xuất](/images/2/2-2.png)

{{% notice tip %}}
Trước khi mua, hãy dùng **Purchase Analyzer** trong cùng menu để thử một mức cam kết: công cụ này ước tính chi phí, độ bao phủ (coverage) và mức sử dụng (utilization) của plan dựa trên phần sử dụng thực tế của bạn.
{{% /notice %}}
