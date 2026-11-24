---
title : "Xem xét hiệu năng"
weight : 3
chapter : false
pre : " <b> 3.3. </b> "
---

#### Tổng quan

**Tải cơ sở dữ liệu** được đo bằng **average active sessions (AAS)**: số phiên trung bình đang chạy hoặc đang chờ cùng lúc. Database Insights chia tải theo **wait event**, để bạn thấy các phiên đang tốn thời gian vào việc gì:

| Wait event | Ý nghĩa |
|---|---|
| `CPU` | Phiên đang chạy trên CPU hoặc đang chờ CPU |
| `IO:DataFileRead` | Phiên đọc một trang không có trong bộ nhớ (shared buffers) |
| `IO:WalSync`, `LWLock:WALWrite` | Phiên chờ ghi nhật ký giao dịch (WAL) khi `COMMIT` |
| `Lock:transactionid`, `Lock:tuple` | Phiên chờ một dòng mà giao dịch khác đang cập nhật |

Khi tải cao hơn số vCPU (đường **Max vCPUs**), các phiên phải chờ nhau.

#### Xem tải cơ sở dữ liệu

1. Mở [CloudWatch console](https://console.aws.amazon.com/cloudwatch/), chọn **Database Insights**, rồi chọn **Database Instance** và `rdspg-lab-db`. Chọn khoảng thời gian **30m**:
    - (1) Trong lúc kiểm tra tải, tải cơ sở dữ liệu lên khoảng 45 phiên hoạt động, gần như toàn màu xanh lá.
    - (2) Màu xanh lá là wait event `CPU`. Đường nét đứt **Max vCPUs** ở mức 2, bằng số vCPU của `db.t4g.micro`.

    ![Database load](/images/3/3-3/1.png)

2. Kéo xuống tab **DB Load Analysis**:
    - (1) Mở **Top SQL**.
    - (2) Câu lệnh có tải cao nhất là `UPDATE pgbench_accounts ...`, câu cập nhật bảng lớn nhất. Phần lớn tải của nó cũng là `CPU`.

    ![Top SQL](/images/3/3-3/2.png)

{{% notice tip %}}
Dùng **Top waits**, **Top hosts**, **Top users** và **Top databases** để chia cùng một tải theo các chiều khác. Ví dụ, **Top hosts** cho thấy mọi phiên đều đến từ địa chỉ IP private của App server.
{{% /notice %}}

#### Kiểm tra alarm

3. Kéo lên phần **Alarms**:
    - (1) `awsrds-rdspg-lab-db-High-CPUUtilization` ở trạng thái **In alarm**: mức sử dụng CPU lên khoảng 95%, vượt ngưỡng 50%.

    ![Alarm in Database Insights](/images/3/3-3/3.png)

Hãy kiểm tra hộp thư. Amazon SNS gửi thư *ALARM: "awsrds-rdspg-lab-db-High-CPUUtilization"* trong vòng một đến hai phút sau khi trạng thái thay đổi.

#### Kết luận

Cơ sở dữ liệu dành gần như toàn bộ thời gian cho CPU, và số phiên hoạt động nhiều hơn hẳn số vCPU. Điểm nghẽn là CPU, không phải I/O lưu trữ hay khóa. Ở bước tiếp theo bạn cấp cho cơ sở dữ liệu nhiều vCPU và bộ nhớ hơn.
