---
title : "Kết nối RDS bằng psql từ App server"
weight : 3
chapter : false
pre : " <b> 3.3. </b> "
---

#### Tổng quan

Ở bước này, bạn kết nối tới `rdspg-lab-db` bằng PostgreSQL client `psql` từ `rdspg-app-server`. Bước này kiểm chứng hai phần thiết kế ở chương 2:

| Thiết kế | Cách kiểm chứng |
|---|---|
| App server nằm trong private subnet, không có NAT gateway | Không ra được internet, nhưng vẫn cài được `psql` qua **S3 Gateway endpoint** |
| `rds-sg` cho phép cổng 5432 từ `app-sg` | `psql` kết nối được, và RDS thấy client là IP private của App server |

App server không có địa chỉ IP công khai, nên bạn truy cập qua bastion host bằng SSH **ProxyJump**. Private key vẫn nằm trên máy bạn. Không chép private key lên bastion host.

#### Kết nối tới App server qua bastion host

1. Trên máy tính của bạn, tạo file cấu hình SSH. Thay `<BASTION_PUBLIC_IP>` bằng địa chỉ public IPv4 của `rdspg-bastion-host`, và `<APP_SERVER_PRIVATE_IP>` bằng địa chỉ private IPv4 của `rdspg-app-server`.

    ```powershell
    @"
    Host rdspg-bastion
        HostName <BASTION_PUBLIC_IP>
        User ec2-user
        IdentityFile ~/.ssh/rdspg-lab-key.pem

    Host rdspg-app
        HostName <APP_SERVER_PRIVATE_IP>
        User ec2-user
        IdentityFile ~/.ssh/rdspg-lab-key.pem
        ProxyJump rdspg-bastion
    "@ | Set-Content -Path "$HOME\.ssh\config" -Encoding ascii
    ```

{{% notice note %}}
Tạo file bằng PowerShell thay vì Notepad. Notepad có thể lưu thành `config.txt`, SSH sẽ không đọc file này.
{{% /notice %}}

2. Kết nối tới App server. Gõ `yes` khi SSH hỏi xác nhận host key.

    ```powershell
    Get-Content $HOME\.ssh\config
    ssh rdspg-app
    ```

    SSH kết nối tới bastion host trước, sau đó tới App server. Dấu nhắc hiển thị địa chỉ IP private của App server.

    ![SSH to the App server](/images/3-connect/3.3/1.png)

#### Cài đặt PostgreSQL client

3. Trên App server, kiểm tra truy cập internet, liệt kê các phiên bản PostgreSQL client có sẵn, và cài client cùng major version với RDS.

    ```bash
    curl -m 5 -sI https://www.google.com || echo "NO INTERNET"
    sudo dnf list 'postgresql1?'
    sudo dnf install -y postgresql18
    psql --version
    ```

    App server không ra được internet, nhưng `dnf` vẫn cài được `postgresql18` vì kho phần mềm của Amazon Linux 2023 nằm trên Amazon S3.

    ![Install psql](/images/3-connect/3.3/2.png)

#### Chép bộ chứng chỉ RDS

Để xác thực chứng chỉ máy chủ của RDS (`sslmode=verify-full`), `psql` cần bộ chứng chỉ `global-bundle.pem`. App server không tải được từ internet, nên bạn tải trên máy tính của mình rồi chép lên bằng `scp` qua bastion host.

4. Trên máy tính của bạn, chạy:

    ```powershell
    Invoke-WebRequest https://truststore.pki.rds.amazonaws.com/global/global-bundle.pem -OutFile "$HOME\Downloads\global-bundle.pem"
    scp "$HOME\Downloads\global-bundle.pem" rdspg-app:~/
    ```

    ![Copy the certificate bundle](/images/3-connect/3.3/3.png)

#### Kết nối RDS bằng psql

5. Trên App server, kết nối tới database. Thay endpoint bằng endpoint của bạn và nhập master password khi được hỏi.

    ```bash
    export RDSHOST="<RDS_ENDPOINT>"
    psql "host=$RDSHOST port=5432 dbname=postgres user=postgres sslmode=verify-full sslrootcert=./global-bundle.pem"
    ```

6. Kiểm tra các địa chỉ IP mà kết nối sử dụng:

    ```sql
    SELECT inet_server_addr() AS rds_ip, inet_client_addr() AS client_ip;
    ```

    Kết nối dùng **TLSv1.3**. `rds_ip` là IP private của RDS, `client_ip` là IP private của App server.

    ![psql connected](/images/3-connect/3.3/4.png)

7. Hiển thị thông tin kết nối:

    ```sql
    \conninfo
    ```

    ![Connection information](/images/3-connect/3.3/5.png)

| Trường | Giá trị | Ý nghĩa |
|---|---|---|
| Host Address | `10.0.130.72` | Endpoint trỏ về một IP private bên trong VPC |
| SSL Connection | `true` | Kết nối được mã hóa và chứng chỉ máy chủ đã được xác thực |
| Superuser | `off` | Master user không phải superuser của PostgreSQL. Trên Amazon RDS, user này mang role `rds_superuser`, còn superuser thật do AWS giữ để quản lý instance |

8. Gõ `\q` để thoát `psql`, sau đó `exit` để rời App server.

{{% notice tip %}}
Trong `psql`, mỗi dòng chỉ chạy một meta-command như `\conninfo`. Nếu dán nhiều dòng cùng lúc, `psql` sẽ hiểu phần chữ phía sau là tham số của meta-command.
{{% /notice %}}

#### Xử lý sự cố

| Lỗi | Nguyên nhân | Cách xử lý |
|---|---|---|
| `Connection timed out` khi kết nối tới bastion host | Địa chỉ IP công khai của bạn đã thay đổi | Sửa inbound rule của `bastion-sg` và chọn lại **My IP** |
| `Could not resolve hostname rdspg-app` | Bạn chạy lệnh trên EC2 thay vì trên máy tính của mình | Kiểm tra dấu nhắc. Chạy `ssh rdspg-app` từ PowerShell trên máy của bạn |
| SSH chạy được lần đầu, nhưng lỗi sau khi stop rồi start bastion host | Địa chỉ IP công khai của bastion host đã thay đổi | Cập nhật `HostName` của `rdspg-bastion` trong `$HOME\.ssh\config` |
