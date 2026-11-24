---
title : "CloudWatch Alarms"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---


Trong phần này, bạn tạo alarm trên `PythonAppErrorCount`. Khi ứng dụng ghi hơn 5 lỗi trong một phút, alarm chuyển sang **In alarm** và gửi email qua Amazon SNS.

#### Tạo alarm

1. Ở thanh điều hướng CloudWatch chọn **Alarms**, **All alarms**, rồi **Create alarm** (1).

![Create alarm](/images/5-alarm/5-create-alarm.png)

2. Giữ **Metrics** và **Classic**, rồi chọn **Select metric** (1).

![Select metric](/images/5-alarm/5-select-metric.png)

3. Chọn custom namespace **FCJ/CloudWatchWorkshop** (1), rồi **Metrics with no dimensions**.

![Namespace](/images/5-alarm/5-namespace.png)

4. Chọn **PythonAppErrorCount** (1) rồi chọn **Select metric** (2).

![PythonAppErrorCount](/images/5-alarm/5-metric-selected.png)

5. Đổi **Statistic** thành **Sum** (1) và **Period** thành **1 minute** (2). Biểu đồ giờ hiển thị số lỗi mỗi phút.

![Statistic và period](/images/5-alarm/5-stat-period.png)

6. Ở **Conditions** giữ **Static** và **Greater** (1), nhập ngưỡng `5` (2) rồi chọn **Next** (3). Đường đỏ nét đứt trong biểu đồ là ngưỡng.

![Ngưỡng](/images/5-alarm/5-threshold.png)

7. Ở **Notification**, giữ **In alarm** và chọn **Create new topic** (1). Nhập tên topic `FCJ-CloudWatch-Alarm-Topic` (2), địa chỉ email của bạn (3), rồi chọn **Create topic** (4).

![Tạo SNS topic](/images/5-alarm/5-new-topic.png)

8. Topic mới được chọn (1) với email của bạn là endpoint (2). Cuộn xuống và chọn **Next**.

![Đã tạo topic](/images/5-alarm/5-topic-created.png)

9. Nhập **Alarm name** `PythonAppErrorAlarm` (1) và mô tả, ví dụ `More than 5 ERROR lines from python-app in 1 minute` (2). Chọn **Next**.

![Tên alarm](/images/5-alarm/5-alarm-name.png)

10. Kiểm tra lại alarm, xem phần thông báo (1) rồi chọn **Create alarm** (2).

![Create alarm](/images/5-alarm/5-preview-create.png)

11. Alarm được tạo (1). Trạng thái là **Insufficient data** cho đến khi có đủ dữ liệu, và có thông báo nhắc subscription SNS đang chờ xác nhận.

![Đã tạo alarm](/images/5-alarm/5-alarm-created.png)

#### Xác nhận đăng ký email

12. Mở email **AWS Notification - Subscription Confirmation** từ `no-reply@sns.amazonaws.com` và chọn **Confirm subscription** (1).

![Email xác nhận](/images/5-alarm/5-email-confirm.png)

13. Ở trang vừa mở, chọn **Confirm Subscription** (1).

![Confirm subscription](/images/5-alarm/5-sns-confirm.png)

14. Subscription đã được xác nhận (1). SNS chỉ gửi thông báo tới các subscription đã xác nhận.

![Subscription confirmed](/images/5-alarm/5-sns-confirmed.png)

#### Kích hoạt alarm

15. `logger.py` thường ghi 2 đến 5 lỗi mỗi phút nên phần lớn thời gian alarm ở trạng thái **OK**. Hãy giả lập sự cố: mở lại Session Manager tới Instance A và ghi 30 dòng lỗi cùng lúc (1):

   ```bash
   for i in $(seq 1 30); do logger -t python-app "ERROR GET /api/orders failed: simulated outage $i"; done; echo done
   ```

![Giả lập lỗi](/images/5-alarm/5-simulate-errors.png)

16. Chờ 1 đến 3 phút rồi mở alarm. Biểu đồ có đỉnh vượt ngưỡng (1) và dòng thời gian trạng thái chuyển đỏ, tức **In alarm** (2). Sang phút tiếp theo số lỗi giảm và alarm quay về **OK**.

![Alarm được kích hoạt](/images/5-alarm/5-alarm-triggered.png)

17. Kiểm tra hộp thư. SNS đã gửi email **ALARM: "PythonAppErrorAlarm" in ...** (1). Email nêu lý do, giá trị vượt ngưỡng (2) và lần đổi trạng thái `OK -> ALARM` (3).

![Email cảnh báo](/images/5-alarm/5-alarm-email.png)
