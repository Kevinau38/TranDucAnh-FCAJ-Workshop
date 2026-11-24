---
title : "Mua Reserved Instance"
weight : 2
chapter : false
pre : " <b> 4.2. </b> "
---

#### Tổng quan

Bạn tìm một Standard Reserved Instance cho instance Linux `t3.micro` ở Region Singapore, thêm vào giỏ hàng rồi xoá khỏi giỏ.

{{% notice warning %}}
Workshop này không bao giờ hoàn tất việc mua. Savings Plans, Reserved Instances và reserved DB instance là cam kết 1 hoặc 3 năm: khi đã gửi đơn, bạn trả tiền cho mọi giờ trong thời hạn dù có dùng hay không. Bạn chỉ đi đến giỏ hàng, rồi xoá giỏ hàng.
{{% /notice %}}

1. Mở [Amazon EC2 console](https://console.aws.amazon.com/ec2/home#ReservedInstances:) ở Region **Asia Pacific (Singapore)**:
    - (1) Ở menu bên trái, chọn **Reserved Instances**. Danh sách hiển thị các RI của Region: đang hoạt động, đang chờ và đã hết hạn.
    - (2) Banner nhắc rằng AWS khuyến nghị dùng Savings Plans thay cho Reserved Instances.
    - (3) Chọn **Purchase Reserved Instances**.

    ![Reserved Instances](/images/4/4-1.png)

2. Trong **Purchase Reserved Instances**:
    - (1) **Platform**: Linux/UNIX, **Tenancy**: Default, **Offering class**: Standard, **Instance type**: `t3.micro`. Để **Term** và **Payment option** là **Any**.
    - (2) Chọn **Search**.
    - (3) Các offering: thời hạn, giá hiệu dụng (effective rate), giá trả trước, giá theo giờ, hình thức thanh toán và normalized units mỗi giờ. Cột **Seller** là **AWS**, hoặc bên thứ ba với các offering từ Reserved Instance Marketplace.
    - (4) Với offering **12 months, No upfront**, chọn **Add to cart**.

    {{% notice tip %}}
    Bật **Only show offerings that reserve capacity** để tìm zonal RI, loại giữ chỗ capacity trong một Availability Zone do bạn chọn.
    {{% /notice %}}

    ![Tìm offering](/images/4/4-2.png)

3. (1) Giỏ hàng có 1 Reserved Instance, phải trả ngay 0,00 USD cùng chi phí định kỳ trung bình mỗi tháng. (2) Chọn **View cart**.

    ![Tóm tắt giỏ hàng](/images/4/4-3.png)

4. Trong **Shopping cart**:
    - (1) Mục cần đặt, offering class, Region, giá và **Order on** (**Now**, hoặc ngày trong tương lai để xếp hàng chờ).
    - (2) Chọn **Delete** để xoá mục khỏi giỏ hàng.
    - (3) **Order all** sẽ gửi đơn và tính ngay các khoản phí trả trước. **Không chọn nút này.**
    - (4) Dòng ghi chú dưới các nút xác nhận điều đó.

    ![Giỏ hàng](/images/4/4-4.png)

5. Giỏ hàng đã trống. Đóng hộp thoại.

{{% notice info %}}
Sau khi mua, RI chuyển từ **payment-pending** sang **active**. Đơn có ngày **Order on** trong tương lai sẽ ở trạng thái **queued**; bạn có thể xoá RI đang chờ trước ngày đó (**Actions** > **Delete queued Reserved Instances**). RI đang hoạt động không thể huỷ, nhưng bạn có thể bán Standard RI trên Reserved Instance Marketplace hoặc exchange Convertible RI.
{{% /notice %}}
