---
title : "Tạo Security Group cho EC2"
weight : 2
chapter : false
pre : " <b> 2.2. </b> "
---

#### Tổng quan

Ở bước này, bạn tạo hai security group cho các EC2 instance:

| Security group | Gắn vào | Inbound rule |
|---|---|---|
| `bastion-sg` | Bastion host | SSH (22) từ **My IP** |
| `app-sg` | App server | SSH (22) từ **bastion-sg** |

Chỉ máy tính của bạn kết nối được tới bastion host, và chỉ bastion host kết nối được tới App server.

{{% notice info %}}
`app-sg` dùng **một security group khác làm nguồn** thay vì địa chỉ IP. Mọi instance được gắn `bastion-sg` đều được phép kết nối, kể cả khi địa chỉ IP của instance đó thay đổi.
{{% /notice %}}

#### Tạo bastion-sg

1. Mở [Amazon EC2 console](https://console.aws.amazon.com/ec2/), chọn **Security Groups** trong mục **Network & Security**, sau đó chọn **Create security group**.

    ![Create security group](/images/2-preparation/2.2/1.png)

2. Trong **Basic details**:
    - (1) **Security group name**: `bastion-sg`
    - (2) **Description**: `Allow SSH from my IP to bastion host`
    - (3) **VPC**: `rdspg-lab-vpc`

    ![bastion-sg basic details](/images/2-preparation/2.2/2.png)

{{% notice warning %}}
Ô **VPC** mặc định là VPC default. Hãy chắc chắn bạn chọn `rdspg-lab-vpc`. Tên và mô tả của security group không thể sửa sau khi tạo.
{{% /notice %}}

3. Trong **Inbound rules**, chọn **Add rule**: (1) **Type**: `SSH`, (2) **Source type**: `My IP`.

    ![bastion-sg inbound rule](/images/2-preparation/2.2/3.png)

4. (1) Giữ mặc định **Outbound rules**. (2) Chọn **Create security group**.

    ![bastion-sg outbound rule](/images/2-preparation/2.2/4.png)

5. `bastion-sg` đã được tạo.

    ![bastion-sg created](/images/2-preparation/2.2/5.png)

{{% notice tip %}}
Địa chỉ IP công khai của bạn có thể thay đổi. Nếu sau này không SSH được vào bastion host, hãy sửa inbound rule này và chọn lại **My IP**.
{{% /notice %}}

#### Tạo app-sg

6. Chọn **Create security group** một lần nữa. Trong **Basic details**:
    - (1) **Security group name**: `app-sg`
    - (2) **Description**: `Allow SSH from bastion host to app server`
    - (3) **VPC**: `rdspg-lab-vpc`

    ![app-sg basic details](/images/2-preparation/2.2/6.png)

7. Trong **Inbound rules**, chọn **Add rule**: (1) **Type**: `SSH`, (2) **Source type**: `Custom`, (3) **Source**: chọn `bastion-sg`.

    ![app-sg inbound rule](/images/2-preparation/2.2/7.png)

8. Giữ mặc định outbound rules và chọn **Create security group**. Inbound rule của `app-sg` chỉ cho phép SSH từ `bastion-sg`.

    ![app-sg created](/images/2-preparation/2.2/8.png)
