---
title : "Cài đặt pgAdmin 4"
weight : 1
chapter : false
pre : " <b> 3.1. </b> "
---

#### Tổng quan

[pgAdmin 4](https://www.pgadmin.org/) là công cụ mã nguồn mở phổ biến để quản trị và phát triển cơ sở dữ liệu PostgreSQL. Ở bước này, bạn cài pgAdmin 4 trên máy tính của mình. Ở bước tiếp theo, bạn dùng nó để kết nối tới `rdspg-lab-db` thông qua bastion host.

| Khu vực | Công dụng |
|---|---|
| **Object Explorer** | Duyệt server, database, schema và table dạng cây thư mục |
| Các tab **Dashboard, Properties, SQL** | Xem chi tiết đối tượng đang chọn trong Object Explorer |
| **Add New Server** | Thêm kết nối tới một PostgreSQL server |
| **Query Tool** | Soạn và chạy câu lệnh SQL |

#### Tải pgAdmin 4

1. Mở [trang tải pgAdmin 4 cho Windows](https://www.pgadmin.org/download/pgadmin-4-windows/):
    - (1) Chọn phiên bản mới nhất. Workshop này dùng **pgAdmin 4 v9.18**.

    ![pgAdmin download page](/images/3-connect/3.1/1.png)

2. Ở trang danh sách file:
    - (1) Chọn `pgadmin4-9.18-x64.exe` để tải bộ cài đặt.

    ![pgAdmin installer file](/images/3-connect/3.1/2.png)

{{% notice note %}}
Bộ cài được ký số bởi trưởng dự án pgAdmin (**Open Source Developer, David John Page**). Nếu Windows SmartScreen hiện cảnh báo do không kết nối được, hãy kiểm tra thông tin Publisher và chọn **Run**.
{{% /notice %}}

#### Cài đặt pgAdmin 4

3. Chạy bộ cài. Tại **Select install mode**:
    - (1) Chọn **Install for me only (recommended)**.

    ![Select install mode](/images/3-connect/3.1/3.png)

{{% notice info %}}
**Install for me only** không cần quyền administrator. **Install for all users** cài pgAdmin cho mọi tài khoản trên máy và cần quyền administrator.
{{% /notice %}}

4. Giữ các tùy chọn mặc định ở các màn hình còn lại, sau đó chọn **Install** và **Finish**.

#### Kiểm tra cài đặt

5. Mở **pgAdmin 4** từ Start menu. Trong **Object Explorer**, mục **Servers** đang trống vì bạn chưa thêm server nào.

    ![pgAdmin 4](/images/3-connect/3.1/4.png)

{{% notice tip %}}
Nếu máy bạn đã cài sẵn PostgreSQL, pgAdmin có thể tự thêm các server cục bộ đó vào **Servers**. Để danh sách gọn gàng, chuột phải từng server và chọn **Remove Server**. Thao tác này chỉ xóa server khỏi danh sách của pgAdmin, không gỡ cài đặt PostgreSQL hay xóa dữ liệu.
{{% /notice %}}
