---
title : "Tạo hoạt động cho cơ sở dữ liệu"
weight : 2
chapter : false
pre : " <b> 3.2. </b> "
---

#### Tổng quan

`pgbench` chạy song song một giao dịch nhỏ rất nhiều lần và báo số giao dịch mỗi giây (tps) mà cơ sở dữ liệu hoàn thành. Giao dịch mặc định, tương tự benchmark TPC-B, cập nhật ba bảng và thêm một dòng vào bảng thứ tư:

| Bảng | Số dòng với scale 1 | Số dòng với scale 50 |
|---|---|---|
| `pgbench_branches` | 1 | 50 |
| `pgbench_tellers` | 10 | 500 |
| `pgbench_accounts` | 100.000 | 5.000.000 |
| `pgbench_history` | 0 | Tăng theo mỗi giao dịch |

Bạn chạy mọi lệnh trên App server. `PGHOST`, `PGUSER` và `~/.pgpass` từ bước Chuẩn bị giúp `pgbench` kết nối mà không hỏi mật khẩu.

#### Tạo dữ liệu thử

1. Kết nối vào App server (`ssh rdspg-app`), rồi chạy:

    ```bash
    time pgbench -i --fillfactor=90 --scale=50 pglab
    ```

    - (1) `pgbench` xóa các bảng thử cũ, tạo bốn bảng, sinh dữ liệu, chạy `VACUUM` và tạo khóa chính.
    - (2) Với scale 50, dữ liệu thử khoảng 750 MB và mất chưa đến một phút để tạo.

    ![pgbench initialization](/images/3/3-2/1.png)

#### Chạy kiểm tra tải

2. Chạy kiểm tra tải 5 phút với 50 client đồng thời:

    ```bash
    pgbench --protocol=prepared -P 30 --time=300 --client=50 --jobs=2 pglab
    ```

    - (1) Cứ 30 giây, `pgbench` in thông lượng và độ trễ của khoảng thời gian vừa qua.
    - (2) Cuối cùng, `pgbench` in phần tổng kết. Trên `db.t4g.micro`, kết quả khoảng **337 tps** với độ trễ trung bình khoảng **148 ms**.

    ![Load test on db.t4g.micro](/images/3/3-2/2.png)

{{% notice note %}}
Số liệu của bạn có thể khác, vì `db.t4g.micro` là instance dạng burstable: hiệu năng phụ thuộc vào số CPU credit instance đã tích lũy. Hãy ghi lại kết quả, bạn sẽ so sánh với lần kiểm tra tải thứ hai ở bước 3.4.
{{% /notice %}}

Trong lúc kiểm tra tải đang chạy, hãy chuyển sang bước tiếp theo để theo dõi cơ sở dữ liệu trong Database Insights.

{{% notice warning %}}
`db.t4g.micro` cho phép khoảng 80 kết nối. Đừng đặt `--client` cao hơn khoảng 70, nếu không `pgbench` sẽ lỗi *too many clients already*.
{{% /notice %}}
