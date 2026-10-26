---
title : "Tạo EC2 instance cho App Server"
weight : 5
chapter : false
pre : " <b> 2.5. </b> "
---

#### Tổng quan

Ở bước này, bạn tạo `rdspg-app-server`, một EC2 instance nằm trong **private subnet**. Ở các bước sau, bạn cài PostgreSQL client (`psql`) lên máy này và kết nối tới Amazon RDS.

| Thiết lập | Giá trị | Lý do |
|---|---|---|
| AMI | Amazon Linux 2023 | Amazon Linux 2 đã hết hỗ trợ từ ngày 30/6/2026 |
| Instance type | `t3.micro` | Đủ cho một PostgreSQL client |
| Subnet | `rdspg-lab-subnet-private1-ap-southeast-1a` | App server không cần truy cập được từ internet |
| Public IP | Tắt | Instance trong private subnet không cần IP công khai |
| Security group | `app-sg` | Chỉ cho phép SSH từ `bastion-sg` |

Bạn cũng tạo key pair `rdspg-lab-key`, dùng lại cho bastion host ở bước sau.

#### Tạo instance

1. Mở [Amazon EC2 console](https://console.aws.amazon.com/ec2/):
    - (1) Chọn **Instances**.
    - (2) Chọn **Launch instances**.

    ![Launch instances](/images/2-preparation/2.5/1.png)

2. Cấu hình tên và image:
    - (1) **Name**: `rdspg-app-server`
    - (2) **Amazon Machine Image (AMI)**: **Amazon Linux 2023 kernel-6.18 AMI**. Giữ **Architecture** là **64-bit (x86)**.

    ![Name and AMI](/images/2-preparation/2.5/2.png)

3. Cấu hình instance type và key pair:
    - (1) **Instance type**: `t3.micro`
    - (2) Chọn **Create new key pair**.

    ![Instance type and key pair](/images/2-preparation/2.5/3.png)

4. Trong **Create key pair**:
    - (1) **Key pair name**: `rdspg-lab-key`
    - (2) **Key pair type**: **RSA**
    - (3) **Private key file format**: **.pem**
    - (4) Chọn **Create key pair**.

    ![Create key pair](/images/2-preparation/2.5/4.png)

{{% notice warning %}}
Trình duyệt sẽ tải về file `rdspg-lab-key.pem`. Hãy lưu file ở nơi an toàn, bên ngoài thư mục workshop. AWS không cho tải lại private key, và bạn cần file này để kết nối tới các instance.
{{% /notice %}}

5. Trong **Network settings**, chọn **Edit**:
    - (1) **VPC**: `rdspg-lab-vpc`
    - (2) **Subnet**: `rdspg-lab-subnet-private1-ap-southeast-1a`
    - (3) **Auto-assign public IP**: **Disable**
    - (4) **Firewall (security groups)**: chọn **Select existing security group**, sau đó chọn `app-sg`.
    - (5) Chọn **Launch instance**.

    ![Network settings](/images/2-preparation/2.5/5.png)

{{% notice tip %}}
Subnet hiển thị **4091** địa chỉ IP khả dụng. Subnet `/20` có 4.096 địa chỉ, và AWS giữ lại 5 địa chỉ trong mỗi subnet.
{{% /notice %}}

6. Instance đã được khởi tạo.

    ![Launch success](/images/2-preparation/2.5/6.png)

#### Kiểm tra instance

7. Mở instance `rdspg-app-server` và kiểm tra:
    - (1) **Instance state**: **Running**
    - (2) **Private IPv4 address**: một địa chỉ thuộc `10.0.128.0/20`
    - (3) **Public IPv4 address**: trống

    ![Instance summary](/images/2-preparation/2.5/7.png)
