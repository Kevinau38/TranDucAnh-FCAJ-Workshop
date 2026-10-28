---
title : "Tạo RDS PostgreSQL DB"
weight : 7
chapter : false
pre : " <b> 2.7. </b> "
---

#### Tổng quan

Ở bước này, bạn tạo `rdspg-lab-db`, một Amazon RDS for PostgreSQL DB instance nằm trong private subnet, sử dụng mạng và các security group đã tạo ở các bước trước.

| Thiết lập | Giá trị | Lý do |
|---|---|---|
| Cách tạo | Full configuration | Tự chọn VPC, DB subnet group và security group |
| Engine version | PostgreSQL 18.6-R1 | Phiên bản mới nhất tại thời điểm cập nhật workshop |
| Template | Sandbox | Giới hạn lựa chọn ở các cấu hình chi phí thấp, phục vụ học tập |
| Deployment | Single-AZ DB instance | Workshop không cần instance standby |
| Instance class | `db.t4g.micro` | Thuộc gói miễn phí, dùng chip Graviton |
| Storage | gp3, 20 GiB, tắt autoscaling | Dung lượng tối thiểu, đủ cho workshop |
| Credentials | Self managed | Secrets Manager tính phí cho mỗi secret hằng tháng |
| Compute resource | Don't connect to an EC2 compute resource | Bạn đã tạo `rds-sg` ở bước 2.3 |
| Public access | No | Database chỉ truy cập được từ bên trong VPC |
| Security group | Chỉ `rds-sg` | Chỉ cho phép cổng 5432 từ `app-sg` và `bastion-sg` |
| Deletion protection | Tắt | Để xóa được database ở bước dọn dẹp tài nguyên |

#### Chọn engine và template

1. Mở [Amazon RDS console](https://console.aws.amazon.com/rds/):
    - (1) Chọn **Databases**.
    - (2) Chọn **Create database**.
    - (3) Chọn **Full configuration**.

    ![Create database](/images/2-preparation/2.7/1.png)

2. Chọn engine và template:
    - (1) **Engine type**: **PostgreSQL**
    - (2) **Database creation method**: **Full configuration**
    - (3) **Templates**: **Sandbox**

    ![Engine and template](/images/2-preparation/2.7/2.png)

{{% notice warning %}}
Template mặc định là **Production**, sẽ chọn sẵn Multi-AZ và cấu hình lớn. Hãy chắc chắn bạn chọn **Sandbox**.
{{% /notice %}}

3. Trong **Availability and durability**:
    - (1) Chọn **Single-AZ DB instance deployment (1 instance)**.

    ![Availability and durability](/images/2-preparation/2.7/3.png)

{{% notice info %}}
Với template Sandbox, các lựa chọn Multi-AZ bị vô hiệu hóa. **Multi-AZ DB instance** gồm một primary và một standby không phục vụ đọc. **Multi-AZ DB cluster** gồm một primary và hai standby đọc được. **Single-AZ** không có standby và không có dữ liệu dự phòng, chấp nhận được cho lab nhưng không dùng cho production.
{{% /notice %}}

#### Cấu hình settings và credentials

4. Trong **Settings**:
    - (1) **Engine version**: **PostgreSQL 18.6-R1**
    - (2) Không chọn **Enable RDS Extended Support**.
    - (3) **DB instance identifier**: `rdspg-lab-db`
    - (4) **Master username**: `postgres`

    ![Settings](/images/2-preparation/2.7/4.png)

{{% notice note %}}
**RDS Extended Support** là dịch vụ trả phí giúp tiếp tục chạy một major version sau khi hết thời gian hỗ trợ chuẩn. PostgreSQL 18 còn mới nên bạn không cần.
{{% /notice %}}

5. Trong **Credentials management**:
    - (1) Chọn **Self managed**.
    - (2) Không chọn **Auto generate password**.
    - (3) Nhập **Master password**.
    - (4) Nhập lại mật khẩu ở **Confirm master password**.
    - (5) **Database authentication options**: **Password authentication**

    ![Credentials](/images/2-preparation/2.7/5.png)

{{% notice warning %}}
Lưu master password ở nơi an toàn, bên ngoài thư mục workshop. Bạn cần mật khẩu này để kết nối tới database, và RDS không hiển thị lại. Mật khẩu không được chứa các ký tự `/`, `'`, `"` hoặc `@`.
{{% /notice %}}

#### Cấu hình instance và storage

6. Trong **Instance configuration** và **Storage**:
    - (1) Chọn **Burstable classes (includes t classes)**.
    - (2) **Instance type**: `db.t4g.micro`
    - (3) **Storage type**: **General Purpose SSD (gp3)**
    - (4) **Allocated storage**: `20` GiB

    ![Instance and storage](/images/2-preparation/2.7/6.png)

7. Trong **Additional storage configuration**:
    - (1) Bỏ chọn **Enable storage autoscaling**.

    ![Storage autoscaling](/images/2-preparation/2.7/7.png)

{{% notice tip %}}
Với gp3 dưới 400 GiB, RDS cung cấp sẵn 3.000 IOPS và 125 MiBps mà không tính thêm phí. Storage autoscaling được tắt vì RDS không thể giảm dung lượng sau khi đã tăng.
{{% /notice %}}

#### Cấu hình kết nối

8. Trong **Connectivity**:
    - (1) **Compute resource**: **Don't connect to an EC2 compute resource**
    - (2) **Network type**: **IPv4**
    - (3) **Virtual private cloud (VPC)**: `rdspg-lab-vpc`
    - (4) **DB subnet group**: `rdspg-lab-subnet-group`

    ![Connectivity](/images/2-preparation/2.7/8.png)

{{% notice warning %}}
Bạn không thể đổi VPC sau khi database đã được tạo.
{{% /notice %}}

9. Tiếp tục trong **Connectivity**:
    - (1) **Public access**: **No**
    - (2) **VPC security group (firewall)**: **Choose existing**
    - (3) **Existing VPC security groups**: chỉ chọn `rds-sg`. Xóa `default` nếu đang được chọn.
    - (4) **Availability Zone**: **No preference**

    ![Public access and security group](/images/2-preparation/2.7/9.png)

10. Trong **Additional configuration** của **Connectivity**:
    - (1) Giữ **Database port** là `5432`.

    ![Database port](/images/2-preparation/2.7/10.png)

{{% notice note %}}
Không chọn **Create an RDS Proxy** và giữ **Certificate authority** mặc định. Inbound rule của `rds-sg` cho phép cổng 5432, vì vậy không đổi port.
{{% /notice %}}

#### Cấu hình monitoring và tạo database

11. Trong **Monitoring**:
    - (1) Giữ **Database Insights - Standard**.
    - (2) Giữ **Retention period** là **7 days (free)**.

    ![Monitoring](/images/2-preparation/2.7/11.png)

{{% notice info %}}
Giữ các thiết lập mặc định sau:
- **Enhanced Monitoring**, **Log exports** và **DevOps Guru**: tắt, vì sẽ phát sinh phí.
- **Initial database name**: để trống. PostgreSQL luôn có sẵn database mặc định `postgres`.
- **DB parameter group**: `default.postgres18`. Parameter group chứa cấu hình của toàn bộ DB instance.
- **Encryption**: bật, dùng khóa do AWS quản lý `aws/rds`.
- **Automated backup**: bật, thời gian lưu giữ 1 ngày.
- **Auto minor version upgrade**: bật. RDS tự nâng minor version (ví dụ 18.6 lên 18.7) trong khung giờ bảo trì.
{{% /notice %}}

12. Ở cuối trang:
    - (1) Không chọn **Enable deletion protection**.
    - (2) Chọn **Create database**.

    ![Create database](/images/2-preparation/2.7/12.png)

13. Đợi khoảng 10 đến 15 phút đến khi **Status** của `rdspg-lab-db` là **Available**.

    ![Database available](/images/2-preparation/2.7/13.png)

#### Kiểm tra database

14. Chọn `rdspg-lab-db`, sau đó mở tab **Connectivity & security**:
    - (1) Tại **Connect using**, chọn **Code snippets**.

    Phần **Connection steps** hiển thị endpoint và lệnh `psql` bạn sẽ dùng ở bước sau. **Internet access gateway** là **Disabled**.

    ![Code snippets](/images/2-preparation/2.7/14.png)

15. Tại **Connect using**:
    - (1) Chọn **Endpoints**.

    ![Endpoints](/images/2-preparation/2.7/15.png)

16. Trong **Additional configurations**, kiểm tra:
    - **Endpoint** và **Port** `5432`
    - **VPC**: `rdspg-lab-vpc` và **Subnet group**: `rdspg-lab-subnet-group`
    - **VPC security groups**: `rds-sg`, **Active**
    - **Publicly accessible**: **No**

    ![Connectivity and security](/images/2-preparation/2.7/16.png)

17. Trong **Security group rules**, kiểm tra `rds-sg` có hai inbound rule, một từ `app-sg` và một từ `bastion-sg`.

    ![Security group rules](/images/2-preparation/2.7/17.png)

{{% notice tip %}}
Sao chép **Endpoint** và lưu cùng với master password. Mọi kết nối ở các chương sau đều dùng endpoint này.
{{% /notice %}}

{{% notice warning %}}
Từ bây giờ, RDS instance và hai EC2 instance được tính phí theo giờ. Nếu tạm dừng workshop lâu, hãy stop chúng. RDS sẽ tự khởi động lại DB instance đã stop sau 7 ngày.
{{% /notice %}}
