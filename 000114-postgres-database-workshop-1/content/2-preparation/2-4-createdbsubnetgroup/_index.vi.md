---
title : "Tạo DB Subnet Group"
weight : 4
chapter : false
pre : " <b> 2.4. </b> "
---

#### Tổng quan

**DB subnet group** cho Amazon RDS biết được phép đặt database ở những subnet nào. Khi tạo RDS instance, bạn chọn subnet group chứ không chọn từng subnet.

Ở bước này, bạn tạo `rdspg-lab-subnet-group` gồm **hai private subnet** đã tạo ở bước 2.1:

| Subnet | Availability Zone | CIDR |
|---|---|---|
| `rdspg-lab-subnet-private1-ap-southeast-1a` | `ap-southeast-1a` | `10.0.128.0/20` |
| `rdspg-lab-subnet-private2-ap-southeast-1b` | `ap-southeast-1b` | `10.0.144.0/20` |

{{% notice info %}}
DB subnet group phải có subnet ở **ít nhất hai Availability Zone**, kể cả khi không bật Multi-AZ. Nhờ vậy RDS có thể đặt bản standby hoặc chuyển database sang AZ khác khi cần.
{{% /notice %}}

#### Tạo DB subnet group

1. Mở [Amazon RDS console](https://console.aws.amazon.com/rds/):
    - (1) Chọn **Subnet groups**.
    - (2) Chọn **Create DB subnet group**.

    ![Create DB subnet group](/images/2-preparation/2.4/1.png)

2. Trong **Subnet group details**:
    - (1) **Name**: `rdspg-lab-subnet-group`
    - (2) **Description**: `Private subnets for RDS PostgreSQL`
    - (3) **VPC**: `rdspg-lab-vpc`

    ![Subnet group details](/images/2-preparation/2.4/2.png)

3. Trong **Add subnets**:
    - (1) **Availability Zones**: chỉ giữ `ap-southeast-1a` và `ap-southeast-1b`. Bỏ các AZ khác được chọn sẵn.
    - (2) **Subnets**: chọn hai private subnet `10.0.128.0/20` và `10.0.144.0/20`.
    - (3) Chọn **Create**.

    ![Add subnets](/images/2-preparation/2.4/3.png)

{{% notice warning %}}
Không chọn các public subnet (`10.0.0.0/20` và `10.0.16.0/20`). Database phải nằm trong private subnet.
{{% /notice %}}

{{% notice note %}}
Console hiển thị thông báo **Multi-AZ DB cluster** cần ba subnet ở ba AZ. Workshop này dùng một DB instance nên hai subnet ở hai AZ là đủ.
{{% /notice %}}

4. DB subnet group `rdspg-lab-subnet-group` đã được tạo trong `rdspg-lab-vpc`.

    ![DB subnet group created](/images/2-preparation/2.4/4.png)
