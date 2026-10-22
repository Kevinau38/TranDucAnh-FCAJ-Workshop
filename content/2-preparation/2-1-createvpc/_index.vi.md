---
title : "Tạo VPC"
weight : 1
chapter : false
pre : " <b> 2.1. </b> "
---

#### Tổng quan

Ở bước này, bạn tạo mạng cho toàn bộ workshop: một VPC gồm **2 public subnet** và **2 private subnet**, trải trên **2 Availability Zone (AZ)**.

| Tài nguyên | Subnet | Lý do |
|---|---|---|
| Bastion host | Public | Bạn kết nối vào từ máy tính qua SSH |
| App server | Private | Không cần truy cập được từ internet |
| RDS PostgreSQL | Private (2 AZ) | DB subnet group yêu cầu subnet ở ít nhất 2 AZ |

{{% notice info %}}
Workshop này **không** dùng NAT gateway vì NAT gateway tính phí theo giờ. App server ở private subnet vẫn cài được phần mềm: kho phần mềm của Amazon Linux 2023 nằm trên Amazon S3, và **S3 Gateway endpoint** tạo đường đi từ private subnet tới S3 mà không phát sinh thêm chi phí.
{{% /notice %}}

#### Tạo VPC

1. Mở [Amazon VPC console](https://console.aws.amazon.com/vpc/), chọn **Your VPCs**, sau đó chọn **Create VPC**.

    ![Create VPC](/images/2-preparation/2.1/1.png)

2. Trong **VPC settings**:
    - (1) Tại **Resources to create**, chọn **VPC and more**.
    - (2) Tại **Name tag auto-generation**, giữ **Auto-generate** và nhập `rdspg-lab`.
    - (3) Tại **IPv4 CIDR block**, nhập `10.0.0.0/16`. Tại **IPv6 CIDR block**, chọn **No IPv6 CIDR block**.
    - Giữ **Tenancy** là **Default** và không thay đổi **Encryption settings**.

    ![VPC settings](/images/2-preparation/2.1/2.png)

3. Cấu hình subnet và kết nối:
    - (1) **Number of Availability Zones (AZs)**: `2`
    - (2) **Number of public subnets**: `2`
    - (3) **Number of private subnets**: `2`
    - (4) **NAT gateways**: **None**
    - (5) **VPC endpoints**: **S3 Gateway**

    ![Subnets and connections](/images/2-preparation/2.1/3.png)

4. (1) Tại **DNS options**, giữ chọn cả **Enable DNS hostnames** và **Enable DNS resolution**. (2) Chọn **Create VPC**.

    ![DNS options](/images/2-preparation/2.1/4.png)

{{% notice note %}}
Cả hai tùy chọn DNS đều bắt buộc. Bạn kết nối tới Amazon RDS qua **endpoint**, là một tên miền dạng `xxx.rds.amazonaws.com`, không phải địa chỉ IP.
{{% /notice %}}

5. Đợi đến khi tất cả các bước hiển thị **Success**, sau đó chọn **View VPC**.

    ![Create VPC workflow](/images/2-preparation/2.1/5.png)

#### Kiểm tra mạng bằng Resource map

6. Tại trang chi tiết VPC, mở tab **Resource map** và bật **Show all details**.

    ![Resource map](/images/2-preparation/2.1/6.png)

| Route table | Subnet được gắn | Route | Kết quả |
|---|---|---|---|
| `rdspg-lab-rtb-public` | 2 public subnet | `local`, `0.0.0.0/0` → Internet gateway | Public: ra và vào internet được |
| `rdspg-lab-rtb-private1-...` | Private subnet ở AZ 1a | `local`, Amazon S3 → S3 Gateway endpoint | Private: không ra internet, tới được S3 |
| `rdspg-lab-rtb-private2-...` | Private subnet ở AZ 1b | `local`, Amazon S3 → S3 Gateway endpoint | Private: không ra internet, tới được S3 |
| Main route table | Không có | `local` | Tự động tạo cùng mọi VPC, không dùng trong bài |

{{% notice tip %}}
Một subnet là **public** chỉ vì route table của nó có route tới Internet gateway. Bản thân subnet không có ô nào để đánh dấu "public".
{{% /notice %}}
