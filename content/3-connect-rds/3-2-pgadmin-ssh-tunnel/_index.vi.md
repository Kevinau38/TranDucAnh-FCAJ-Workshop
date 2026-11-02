---
title : "Kết nối RDS bằng pgAdmin 4 qua SSH tunnel"
weight : 2
chapter : false
pre : " <b> 3.2. </b> "
---

#### Tổng quan

`rdspg-lab-db` nằm trong private subnet và có **Public access** là **No**, nên máy tính của bạn không kết nối trực tiếp được. Ở bước này, pgAdmin 4 kết nối tới database qua **SSH tunnel** đi xuyên bastion host:

```
pgAdmin (máy bạn) --SSH :22--> Bastion host (public subnet) --:5432--> RDS (private subnet)
```

pgAdmin mở một cổng tạm trên máy bạn (ví dụ `127.0.0.1:57211`). Dữ liệu gửi vào cổng này được chuyển bên trong kết nối SSH tới bastion host, rồi bastion host chuyển tiếp tới endpoint RDS ở cổng 5432. RDS thấy kết nối đến từ bastion host, và `rds-sg` cho phép kết nối này.

| Tab | Trường | Giá trị |
|---|---|---|
| Connection | Host name/address | **Endpoint** của RDS |
| SSH Tunnel | Tunnel host | **Public IPv4 address** của bastion host |

{{% notice warning %}}
Không nhầm lẫn hai host. **Host name/address** là endpoint RDS. **Tunnel host** là địa chỉ IP công khai của bastion host.
{{% /notice %}}

#### Kiểm tra kết nối SSH trước

Trước khi cấu hình pgAdmin, hãy xác nhận bạn SSH được tới bastion host từ Windows PowerShell.

1. Chuyển key pair vào thư mục `.ssh` và giới hạn quyền truy cập. OpenSSH từ chối private key mà người dùng khác đọc được.

    ```powershell
    mkdir $HOME\.ssh -Force
    Move-Item "<đường-dẫn>\rdspg-lab-key.pem" "$HOME\.ssh\rdspg-lab-key.pem"
    icacls "$HOME\.ssh\rdspg-lab-key.pem" /inheritance:r
    icacls "$HOME\.ssh\rdspg-lab-key.pem" /grant:r "$($env:USERNAME):(R)"
    ```

2. Kết nối tới bastion host. Thay `<BASTION_PUBLIC_IP>` bằng địa chỉ public IPv4 ở bước 2.6. Gõ `yes` khi SSH hỏi xác nhận host key.

    ```powershell
    ssh -i "$HOME\.ssh\rdspg-lab-key.pem" ec2-user@<BASTION_PUBLIC_IP>
    ```

3. Trên bastion host, kiểm tra endpoint trỏ về địa chỉ IP private và cổng 5432 đang mở. Thay `<RDS_ENDPOINT>` bằng endpoint của bạn.

    ```bash
    getent hosts <RDS_ENDPOINT>
    timeout 5 bash -c '</dev/tcp/<RDS_ENDPOINT>/5432' && echo "PORT 5432 OPEN"
    ```

    Lệnh đầu trả về một địa chỉ thuộc `10.0.128.0/20` hoặc `10.0.144.0/20`. Lệnh thứ hai in ra `PORT 5432 OPEN`. Gõ `exit` để thoát bastion host.

{{% notice tip %}}
Nếu SSH báo `Connection timed out`, nhiều khả năng địa chỉ IP công khai của bạn đã thay đổi. Sửa inbound rule của `bastion-sg` và chọn lại **My IP**.
{{% /notice %}}

#### Thêm server trong pgAdmin

4. Mở pgAdmin 4:
    - (1) Tại **Quick Links**, chọn **Add New Server**.

    ![Add New Server](/images/3-connect/3.2/1.png)

5. Trong tab **General**:
    - (1) **Name**: `rdspg-lab-db`

    ![General tab](/images/3-connect/3.2/2.png)

6. Trong tab **Connection**:
    - (1) Chọn tab **Connection**.
    - (2) **Host name/address**: endpoint của RDS
    - (3) **Port**: `5432`
    - (4) **Maintenance database**: `postgres`
    - (5) **Username**: `postgres`
    - (6) **Password**: master password. Giữ tắt **Save password?**.

    ![Connection tab](/images/3-connect/3.2/3.png)

7. Trong tab **SSH Tunnel**:
    - (1) Chọn tab **SSH Tunnel**.
    - (2) Bật **Use SSH tunneling**.
    - (3) **Tunnel host**: địa chỉ public IPv4 của bastion host
    - (4) **Tunnel port**: `22`
    - (5) **Username**: `ec2-user`
    - (6) **Authentication**: **Identity file**
    - (7) **Identity file**: `C:\Users\<tên-người-dùng>\.ssh\rdspg-lab-key.pem`
    - (8) Chọn **Save**.

    ![SSH Tunnel tab](/images/3-connect/3.2/4.png)

{{% notice note %}}
Để trống **Password** của SSH và tắt **Prompt for identity file password?**. Key pair do AWS tạo không có passphrase.
{{% /notice %}}

#### Kiểm tra kết nối

8. Trong **Object Explorer**, mở **rdspg-lab-db** > **Databases**. Tab **Dashboard** hiển thị hoạt động thực tế của database.

    ![Connected](/images/3-connect/3.2/5.png)

| Database | Mô tả |
|---|---|
| `postgres` | Database mặc định PostgreSQL luôn tạo sẵn |
| `rdsadmin` | Dành riêng cho Amazon RDS để quản lý instance. Bạn không kết nối được vào database này |

{{% notice info %}}
Nếu pgAdmin báo **fe_sendauth: no password supplied**, SSH tunnel đã hoạt động nhưng ô **Password** ở tab **Connection** đang trống. Nhập master password rồi lưu lại.
{{% /notice %}}
