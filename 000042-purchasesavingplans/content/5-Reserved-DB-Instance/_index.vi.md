---
title : "Reserved DB instance"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

#### Tổng quan

**Reserved DB instance** là một khoản giảm giá trên hoá đơn cho Amazon RDS hoặc Amazon Aurora, thời hạn 1 hoặc 3 năm. Giống EC2 Reserved Instance, nó không phải là một cơ sở dữ liệu: bạn vẫn tạo DB instance như bình thường, và những instance khớp với reservation sẽ được tính theo giá giảm. Khoản giảm chỉ áp dụng cho giờ chạy instance, không áp dụng cho storage, backup hay I/O.

Để được giảm giá, DB instance phải khớp với reservation về:

- AWS Region
- DB engine (không cần khớp phiên bản engine)
- DB instance class, ví dụ `db.t4g.micro`
- Mô hình license (license-included hoặc bring-your-own-license)

**Size flexibility**: với RDS for PostgreSQL, MySQL, MariaDB, Db2 và Oracle BYOL, khoản giảm áp dụng cho mọi size trong cùng instance class family, cho cả Single-AZ và Multi-AZ, tính theo **normalized unit** (ví dụ `micro` = 0,5 unit, `small` = 1, `large` = 4; Multi-AZ tính gấp đôi). RDS for SQL Server và Oracle license-included phải khớp chính xác.

| Offering type | Thời hạn |
|---|---|
| **No Upfront** | Chỉ 1 năm |
| **Partial Upfront** | 1 hoặc 3 năm |
| **All Upfront** | 1 hoặc 3 năm |

{{% notice warning %}}
Workshop này không bao giờ hoàn tất việc mua. Savings Plans, Reserved Instances và reserved DB instance là cam kết 1 hoặc 3 năm: khi đã gửi đơn, bạn trả tiền cho mọi giờ trong thời hạn dù có dùng hay không. Bạn chỉ đi đến giỏ hàng, rồi xoá giỏ hàng.
{{% /notice %}}

#### Tìm hiểu trang mua

1. Mở [Amazon RDS console](https://console.aws.amazon.com/rds/home#reserved-instances:) ở Region **Asia Pacific (Singapore)**:
    - (1) Ở menu bên trái, chọn **Reserved instances**.
    - (2) Danh sách reservation trong Region, ở đây đang trống.
    - (3) Chọn **Purchase Reserved DB Instance**.

    ![Reserved instances](/images/5/5-1.png)

2. Mở **Engine Type** (1). Danh sách (2) gồm mọi engine hỗ trợ reservation: Aurora MySQL và Aurora PostgreSQL, RDS for MySQL, MariaDB và PostgreSQL, Oracle, SQL Server, Db2 và RDS Custom. Chọn **RDS PostgreSQL**.

    ![Engine type](/images/5/5-2.png)

3. Điền thông tin reservation:
    - (1) **Engine Type**: RDS PostgreSQL.
    - (2) **DB instance class**: gõ `db.t4g.micro` vào ô tìm kiếm rồi chọn.
    - (3) **Deployment options**: **Single-AZ DB instance**.
    - (4) **Term**: **1 year**.
    - (5) **Offering type**: **No Upfront**.

    ![Tuỳ chọn reserved DB instance](/images/5/5-3.png)

4. Cuộn xuống:
    - (1) **Number of DB instances**: 1.
    - (2) **Reserved (ID) - optional**: tên để theo dõi reservation. Để trống.
    - (3) Chọn **Next**. Bước này chỉ mở trang xem lại.

    ![Số lượng DB instance](/images/5/5-4.png)

5. Trang **Review and purchase**:
    - (1) **Purchase summary**: normalized unit mỗi giờ (0,5 với instance micro), phí sử dụng theo giờ, engine, instance class và thời hạn bắt đầu từ hôm nay.
    - (2) **Total one-time payment** là 0,00 USD với No Upfront; phí theo giờ sẽ xuất hiện trên hoá đơn hằng tháng.
    - (3) RDS cảnh báo không có DB instance nào trong tài khoản khớp với reservation: bạn vẫn bị tính phí cho mọi giờ trong thời hạn.
    - (4) Chọn **Cancel**.
    - (5) **Purchase reserved instance** sẽ gửi đơn hàng. **Không chọn nút này.**

    ![Xem lại và mua](/images/5/5-5.png)

{{% notice info %}}
Reserved DB instance không thể huỷ. Nếu bạn xoá DB instance đang dùng reservation, bạn vẫn tiếp tục trả tiền cho reservation, và khoản giảm sẽ chuyển sang DB instance khớp cấu hình khác mà bạn tạo trong cùng Region.
{{% /notice %}}

#### Tổng kết

| | Savings Plans | EC2 Reserved Instances | Reserved DB instance |
|---|---|---|---|
| Bạn cam kết | USD mỗi giờ | Một cấu hình instance | Một DB engine và instance class |
| Phù hợp nhất cho | Phần lớn nhu cầu compute và cơ sở dữ liệu | Giữ chỗ capacity trong một Availability Zone | RDS và Aurora chạy ổn định |
| Thay đổi sau khi mua | Không cần, tự áp dụng | Modify, exchange (Convertible) hoặc bán lại (Standard) | Không thể |

Workshop không mua gì nên không có tài nguyên nào cần dọn dẹp.
