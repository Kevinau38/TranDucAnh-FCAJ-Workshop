---
title : "Tạo database Lightsail"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

**Managed database** của Lightsail chạy MySQL hoặc PostgreSQL thay cho bạn: AWS cài engine, vá lỗi và sao lưu hằng ngày. WordPress lưu bài viết và cấu hình trong database này, vì vậy bạn tạo nó trước. Database cần khoảng 10–15 phút để sẵn sàng, trong lúc đó bạn triển khai instance WordPress.

1. Mở [Lightsail console](https://lightsail.aws.amazon.com/). Ở menu bên trái, chọn **Databases**, rồi chọn **Create database**.

![Trang Databases](/images/1/1-databases.png)

2. Ở mục **Select a Region**, chọn **Singapore (ap-southeast-1)**. Giữ Availability Zone mặc định.

![Chọn Region](/images/1/2-region.png)

3. Ở mục **Pick your database**, chọn **MySQL** và giữ phiên bản 8.4 mới nhất.

![Chọn MySQL](/images/1/3-engine.png)

4. **Không** chọn **Specify login credentials**. Khi đó Lightsail tạo user `dbmasteruser` cùng một mật khẩu mạnh và lưu mật khẩu đó giúp bạn. Ở các chương sau, bạn đọc mật khẩu bằng AWS CLI nên không bao giờ phải chép tay.

5. Chọn **Specify the master database name** và nhập `wordpress`. Lightsail thêm tiền tố `db`, nên tên database là `dbwordpress`.

![Tên master database](/images/1/5-db-name.png)

6. Ở mục **Choose your database plan**, giữ **Standard** và chọn gói **15 USD** (1 GB bộ nhớ, 2 vCPU, 40 GB SSD).

![Gói database](/images/1/4-plan.png)

{{% notice info %}}
Gói **High Availability** giữ một bản dự phòng ở Availability Zone thứ hai và có giá gấp đôi. Gói Standard là đủ cho bài lab.
{{% /notice %}}

7. Ở mục **Identify your database**, nhập `fcj-lightsail-db`. Ở mục **Key-value tags**, thêm tag `Project` = `fcj-lightsail`. Chọn **Create database**.

![Tên và tag](/images/1/6-name-tag.png)

8. Database hiển thị trạng thái **Creating**. Tiếp tục sang chương sau trong lúc chờ.

![Database đang tạo](/images/1/7-creating.png)

9. Khi trạng thái là **Available**, mở database. Tab **Connect** hiển thị user `dbmasteruser`, mật khẩu đang ẩn và endpoint mà WordPress sẽ kết nối tới.

![Thông tin kết nối](/images/1/8-connect.png)

{{% notice note %}}
Mặc định, database Lightsail chỉ nhận kết nối từ các tài nguyên Lightsail của bạn trong cùng Region (**Public mode** đang tắt). Bạn có thể kiểm tra ở tab **Networking**.
{{% /notice %}}
