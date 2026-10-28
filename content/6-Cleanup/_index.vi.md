---
title : "Dọn dẹp tài nguyên"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---

#### Tổng quan

Để tránh phát sinh chi phí ngoài ý muốn, bạn xóa toàn bộ tài nguyên đã tạo trong workshop. RDS DB instance và các EC2 instance bị tính phí khi đang chạy, còn địa chỉ IPv4 public của bastion host bị tính phí theo giờ.

Xóa theo thứ tự dưới đây. Một tài nguyên không xóa được khi tài nguyên khác vẫn đang dùng nó:

| Thứ tự | Tài nguyên | Lý do |
| --- | --- | --- |
| 1 | RDS DB instance `rdspg-lab-db` | Đang dùng DB subnet group và `rds-sg` |
| 2 | EC2 instance `rdspg-bastion-host`, `rdspg-app-server` | Đang dùng `bastion-sg`, `app-sg` và key pair |
| 3 | DB subnet group `rdspg-lab-subnet-group` | Chỉ xóa được sau khi DB instance đã xóa xong |
| 4 | Security group `rds-sg`, `app-sg`, `bastion-sg` | Tham chiếu lẫn nhau trong rule |
| 5 | Key pair `rdspg-lab-key` | |
| 6 | VPC `rdspg-lab-vpc` | Xóa kèm subnet, route table, internet gateway và S3 gateway endpoint |

#### Xóa RDS DB instance

1. Mở [Amazon RDS console](https://console.aws.amazon.com/rds/):
    - (1) Chọn **Databases**.
    - (2) Chọn `rdspg-lab-db`.
    - (3) Chọn **Actions**.
    - (4) Chọn **Delete**.

    ![Delete DB instance](/images/6-cleanup/1.png)

2. Trong hộp thoại **Delete rdspg-lab-db instance**:
    - (1) Bỏ chọn **Create final snapshot**.
    - (2) Bỏ chọn **Retain automated backups**.
    - (3) Chọn ô xác nhận (acknowledge).
    - (4) Nhập `delete me`.
    - (5) Chọn **Delete**.

    ![Confirm DB instance deletion](/images/6-cleanup/2.png)

{{% notice info %}}
Trong môi trường thật, nên giữ **Create final snapshot** để có thể khôi phục dữ liệu sau này. Trong workshop này dữ liệu chỉ là dữ liệu mẫu, và snapshot vẫn bị tính phí lưu trữ sau khi DB instance đã bị xóa.
{{% /notice %}}

DB instance chuyển sang trạng thái **Deleting** và mất vài phút mới biến mất. Trong lúc chờ, bạn làm tiếp phần EC2.

#### Terminate các EC2 instance

1. Mở [Amazon EC2 console](https://console.aws.amazon.com/ec2/):
    - (1) Chọn **Instances**.
    - (2) Chọn `rdspg-app-server` và `rdspg-bastion-host`.
    - (3) Chọn **Instance state**.
    - (4) Chọn **Terminate (delete) instance**.

    ![Terminate instances](/images/6-cleanup/3.png)

2. Trong hộp thoại **Terminate (delete) instances**, để trống ô **Skip OS shutdown**:
    - (1) Chọn **Terminate (delete)**.

    ![Confirm termination](/images/6-cleanup/4.png)

Root EBS volume được xóa cùng với instance.

#### Xóa DB subnet group

Chờ đến khi `rdspg-lab-db` không còn trong danh sách **Databases**. DB subnet group đang được DB instance sử dụng thì không xóa được.

1. Trong Amazon RDS console:
    - (1) Chọn **Subnet groups**.
    - (2) Chọn `rdspg-lab-subnet-group`.
    - (3) Chọn **Delete**.

    ![Delete DB subnet group](/images/6-cleanup/5.png)

2. Trong hộp thoại xác nhận:
    - (1) Chọn **Delete**.

    ![Confirm DB subnet group deletion](/images/6-cleanup/6.png)

#### Xóa các security group

Kiểm tra trước là cả hai EC2 instance đã ở trạng thái **Terminated**. Security group còn gắn với network interface thì không xóa được.

1. Mở [Amazon VPC console](https://console.aws.amazon.com/vpc/):
    - (1) Chọn **Security groups**.
    - (2) Chọn `bastion-sg`.
    - (3) Chọn `app-sg` và `rds-sg`.
    - (4) Chọn **Actions**.
    - (5) Chọn **Delete security groups**.

    ![Delete security groups](/images/6-cleanup/7.png)

2. Hộp thoại cho biết chỉ `rds-sg` sẽ bị xóa. `app-sg` đang được `rds-sg` tham chiếu, còn `bastion-sg` đang được `app-sg` và `rds-sg` tham chiếu:
    - (1) Nhập `delete`.
    - (2) Chọn **Delete**.

    ![Confirm security group deletion](/images/6-cleanup/8.png)

3. Lặp lại các bước trên cho `app-sg`, sau đó cho `bastion-sg`. Mỗi lần xóa sẽ gỡ bỏ tham chiếu đang chặn security group kế tiếp.

{{% notice note %}}
Đây chính là cơ chế bạn đã dùng ở chương 2: security group có thể lấy một security group khác làm nguồn (source) của inbound rule. Vì vậy cần xóa theo thứ tự ngược với chiều tham chiếu: `rds-sg` → `app-sg` → `bastion-sg`.
{{% /notice %}}

#### Xóa key pair

1. Trong Amazon EC2 console:
    - (1) Chọn **Key Pairs**.
    - (2) Chọn `rdspg-lab-key`.
    - (3) Chọn **Actions**.
    - (4) Chọn **Delete**.

    ![Delete key pair](/images/6-cleanup/9.png)

2. Trong hộp thoại xác nhận:
    - (1) Nhập `Delete`.
    - (2) Chọn **Delete**.

    ![Confirm key pair deletion](/images/6-cleanup/10.png)

Xóa key pair trên AWS không xóa file `.pem` trên máy bạn. File này được xóa ở phần cuối.

#### Xóa VPC

1. Trong Amazon VPC console:
    - (1) Chọn **Your VPCs**.
    - (2) Chọn `rdspg-lab-vpc`.
    - (3) Chọn **Actions**.
    - (4) Chọn **Delete VPC**.

    ![Delete VPC](/images/6-cleanup/11.png)

2. Hộp thoại liệt kê các tài nguyên bị xóa kèm theo VPC: internet gateway, các route table, bốn subnet và S3 gateway endpoint.
    - (1) Nhập `delete`.
    - (2) Chọn **Delete**.

    ![Confirm VPC deletion](/images/6-cleanup/12.png)

#### Dọn dẹp trên máy tính

1. Trong pgAdmin, chuột phải vào server `rdspg-lab-db` và chọn **Remove Server**.
2. Xóa hai mục `rdspg-bastion` và `rdspg-app` trong `$HOME\.ssh\config`. Nếu file chỉ có hai mục này thì xóa luôn file.
3. Xóa file private key:

    ```powershell
    Remove-Item "$HOME\.ssh\rdspg-lab-key.pem" -Force
    ```

{{% notice tip %}}
Ngày hôm sau, bạn mở [AWS Billing and Cost Management console](https://console.aws.amazon.com/costmanagement/) để kiểm tra không còn phát sinh chi phí mới cho Amazon RDS, Amazon EC2 hay địa chỉ IPv4 public.
{{% /notice %}}
