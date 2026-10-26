---
title : "Tạo EC2 instance cho Bastion Host"
weight : 6
chapter : false
pre : " <b> 2.6. </b> "
---

#### Tổng quan

Ở bước này, bạn tạo `rdspg-bastion-host`, một EC2 instance nằm trong **public subnet**. Đây là instance duy nhất bạn kết nối tới từ máy tính của mình. Từ bastion host, bạn truy cập App server và Amazon RDS trong các private subnet.

Bastion host dùng các thiết lập giống App server ở bước 2.5, chỉ khác những điểm sau:

| Thiết lập | App server | Bastion host |
|---|---|---|
| Name | `rdspg-app-server` | `rdspg-bastion-host` |
| Subnet | Private (`10.0.128.0/20`) | **Public (`10.0.0.0/20`)** |
| Auto-assign public IP | Disable | **Enable** |
| Security group | `app-sg` | **`bastion-sg`** |
| Key pair | Tạo `rdspg-lab-key` | **Dùng lại `rdspg-lab-key`** |

{{% notice info %}}
Bastion host cần địa chỉ IP công khai để bạn SSH tới từ máy tính. Máy vẫn an toàn vì `bastion-sg` chỉ cho phép SSH từ địa chỉ IP của bạn.
{{% /notice %}}

#### Tạo instance

1. Trong [Amazon EC2 console](https://console.aws.amazon.com/ec2/):
    - (1) Chọn **Instances**.
    - (2) Chọn **Launch instances**.

    ![Launch instances](/images/2-preparation/2.6/1.png)

2. Cấu hình tên và image:
    - (1) **Name**: `rdspg-bastion-host`
    - (2) **Amazon Machine Image (AMI)**: **Amazon Linux 2023 kernel-6.18 AMI**

    ![Name and AMI](/images/2-preparation/2.6/2.png)

3. Cấu hình instance type và key pair:
    - (1) **Instance type**: `t3.micro`
    - (2) **Key pair name**: chọn key pair có sẵn `rdspg-lab-key`.

    ![Instance type and key pair](/images/2-preparation/2.6/3.png)

4. Trong **Network settings**, chọn **Edit**:
    - (1) **VPC**: `rdspg-lab-vpc`
    - (2) **Subnet**: `rdspg-lab-subnet-public1-ap-southeast-1a`
    - (3) **Auto-assign public IP**: **Enable**
    - (4) **Firewall (security groups)**: chọn **Select existing security group**, sau đó chọn `bastion-sg`.
    - (5) Chọn **Launch instance**.

    ![Network settings](/images/2-preparation/2.6/4.png)

5. Instance đã được khởi tạo.

    ![Launch success](/images/2-preparation/2.6/5.png)

#### Kiểm tra instance

6. Mở instance `rdspg-bastion-host` và kiểm tra:
    - (1) **Instance state**: **Running**
    - (2) **Public IPv4 address**: có giá trị
    - (3) **Private IPv4 address**: một địa chỉ thuộc `10.0.0.0/20`

    ![Instance summary](/images/2-preparation/2.6/6.png)

{{% notice tip %}}
Địa chỉ IP công khai được cấp tự động sẽ thay đổi mỗi khi bạn stop rồi start instance. Nếu sau này không SSH được, hãy kiểm tra lại địa chỉ IP công khai hiện tại của bastion host.
{{% /notice %}}
