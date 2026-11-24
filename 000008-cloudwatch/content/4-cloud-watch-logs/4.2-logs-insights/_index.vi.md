---
title : "CloudWatch Logs Insights"
weight : 2
chapter : false
pre : " <b> 4.2 </b> "
---


Đầu tiên bạn chạy ứng dụng mẫu trên Instance A để tạo log, sau đó truy vấn log bằng **Logs Insights**.

#### Chạy ứng dụng mẫu

1. Mở console **EC2**, chọn **Instances**, chọn **FCJ-CloudWatch-Instance-A** (1) rồi chọn **Connect** (2).

![Chọn Instance-A](/images/4-logs/4.2-select-instance.png)

2. Chọn **SSM Session Manager** (1) rồi chọn **Connect** (2).

![Session Manager](/images/4-logs/4.2-connect-ssm.png)

3. Trong terminal, xem script, chạy nền script (1) và kiểm tra script đang chạy (2):

   ```bash
   cat /opt/fcj/logger.py
   nohup python3 /opt/fcj/logger.py > /dev/null 2>&1 &
   ps aux | grep [l]ogger.py
   ```

![Chạy logger.py](/images/4-logs/4.2-logger-start.png)

   Script ghi một dòng sau mỗi 1 đến 4 giây: phần lớn là `INFO`, một số `WARNING` và khoảng 15% là `ERROR`. Hãy để script chạy đến hết bài lab.

4. Xem file log. Các dòng của `python-app` xen giữa các thông báo hệ thống (1)(2). Nhấn **Ctrl+C** để dừng `tail`.

   ```bash
   sudo tail -f /var/log/messages
   ```

![tail /var/log/messages](/images/4-logs/4.2-tail.png)

#### Truy vấn bằng Logs Insights

5. Ở thanh điều hướng CloudWatch chọn **Logs**, **Logs Insights**. Mở bộ chọn log group, lọc theo `/fcj/cloudwatch` (1) rồi chọn **/fcj/cloudwatch-workshop/var/log/messages** (2). Đóng bộ chọn.

![Chọn log group](/images/4-logs/4.2-choose-log-group.png)

6. Kiểm tra log group đã được chọn (1), nhập câu truy vấn dưới đây (2) rồi chọn **Run query** (3).

   ```
   fields @timestamp, @logStream, @message
   | filter @message like /python-app/
   | sort @timestamp desc
   | limit 20
   ```

![Truy vấn Logs Insights](/images/4-logs/4.2-query.png)

7. Biểu đồ và bảng hiển thị 20 dòng mới nhất của `python-app` (1)(2).

![Kết quả truy vấn](/images/4-logs/4.2-results.png)

8. Logs Insights còn có thể tách trường và tổng hợp. Chạy câu truy vấn dưới đây (1)(2). Câu lệnh lấy từ chỉ mức độ ngay sau `python-app[pid]:` và đếm số dòng theo từng mức.

   ```
   filter @message like /python-app/
   | parse @message "python-app[*]: * " as pid, level
   | stats count(*) as events by level
   ```

![Truy vấn stats](/images/4-logs/4.2-stats-query.png)

9. Mở tab **Visualization** và đổi **Graph type** thành **Bar** (1). Bạn thấy số dòng `ERROR`, `WARNING` và `INFO` (2).

![Biểu đồ cột](/images/4-logs/4.2-stats-bar.png)
