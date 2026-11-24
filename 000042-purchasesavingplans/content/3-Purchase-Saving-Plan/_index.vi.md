---
title : "Mua Savings Plan"
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

#### Tổng quan

Trong chương này, bạn cấu hình một Compute Savings Plan trên trang mua, thêm vào giỏ hàng rồi xoá khỏi giỏ. Bạn không gửi đơn hàng.

{{% notice warning %}}
Workshop này không bao giờ hoàn tất việc mua. Savings Plans, Reserved Instances và reserved DB instance là cam kết 1 hoặc 3 năm: khi đã gửi đơn, bạn trả tiền cho mọi giờ trong thời hạn dù có dùng hay không. Bạn chỉ đi đến giỏ hàng, rồi xoá giỏ hàng.
{{% /notice %}}

#### Cấu hình plan

1. Trong menu **Savings Plans**, chọn **Purchase Savings Plans**:
    - (1) **Savings Plans type**: chọn **Compute Savings Plans**.
    - (2) **Term**: chọn **1-year**.

    ![Thông tin mua](/images/3/3-1.png)

2. Điền mức cam kết:
    - (1) **Hourly commitment**: nhập **0.001** (USD mỗi giờ). Số tiền này tính theo giá Savings Plans, không phải giá On-Demand.
    - (2) **Payment option**: chọn **No upfront**.
    - (3) **Start date - optional**: để trống để plan bắt đầu ngay khi gửi đơn, hoặc đặt ngày giờ trong tương lai (UTC) để **xếp hàng chờ (queue)** plan.
    - (4) **Purchase summary** hiển thị chi phí trả trước, khoản trả hằng tháng và tổng chi phí cam kết: 0,001 USD × 8.760 giờ ≈ 8,76 USD cho một năm.
    - (5) Chọn **Add to cart**.

    ![Mức cam kết](/images/3/3-2.png)

3. Vì tài khoản không có mức sử dụng ổn định, AWS cảnh báo bạn có thể đang **cam kết quá mức (overcommitting)**:
    - (1) Cảnh báo so sánh mức cam kết với phần sử dụng của bạn trong 30 ngày qua.
    - (2) Chọn **Confirm add to cart**. Thêm vào giỏ hàng chưa mua gì cả.

    ![Cảnh báo overcommitting](/images/3/3-3.png)

#### Xem lại và xoá giỏ hàng

4. Trang **Cart**:
    - (1) Plan trong giỏ: loại, thời hạn, hình thức thanh toán, ngày bắt đầu, mức cam kết và các khoản thanh toán.
    - (2) **Summary**: tổng cam kết, cam kết bắt đầu ngay, cam kết đang chờ và khoản trả trước phải trả ngay.
    - (3) **Submit order** sẽ mua plan. **Không chọn nút này.**

    ![Giỏ hàng](/images/3/3-4.png)

5. Chọn plan (1), rồi chọn **Remove from cart** (2):

    ![Xoá khỏi giỏ](/images/3/3-5.png)

6. (1) **Your cart is currently empty**, và (2) tổng cam kết trở về 0,00 USD.

    ![Giỏ hàng trống](/images/3/3-6.png)

{{% notice info %}}
Sau khi gửi đơn, bạn có thể **xoá một Savings Plan đang chờ (queued)** (có ngày bắt đầu trong tương lai) trước khi nó bắt đầu. Một plan đang hoạt động có mức cam kết từ 100 USD/giờ trở xuống có thể được **trả lại (return)** trong vòng 7 ngày kể từ khi mua, trong cùng tháng dương lịch. Ngoài các trường hợp trên, cam kết không thể huỷ.
{{% /notice %}}
