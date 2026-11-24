---
title : "Dọn dẹp tài nguyên"
weight : 7
chapter : false
pre : " <b> 7. </b> "
---


Xoá tài nguyên theo thứ tự dưới đây. Dashboard, alarm và SNS topic được tạo trên console nên stack không tự xoá chúng.

1. **Dashboard**: ở **Dashboards**, chọn **FCJ-CloudWatch-Workshop**, chọn **Delete** và xác nhận (1).

![Xoá dashboard](/images/7-cleanup/7-delete-dashboard.png)

2. **Alarm**: ở **Alarms**, chọn **PythonAppErrorAlarm** (1), chọn **Actions**, **Delete** (2) và xác nhận.

![Xoá alarm](/images/7-cleanup/7-delete-alarm.png)

3. **SNS topic**: mở console **Amazon SNS**, **Topics**, chọn **FCJ-CloudWatch-Alarm-Topic**, chọn **Delete**, nhập `delete me` (1) rồi chọn **Delete** (2). Sau đó mở **Subscriptions** và xoá subscription email của topic này nếu vẫn còn.

![Xoá SNS topic](/images/7-cleanup/7-delete-topic.png)

4. **Stack**: ở console **CloudFormation** chọn **FCJ-CloudWatch-Workshop**, chọn **Delete**, nhập tên stack (1) rồi chọn **Delete stack** (2). Stack sẽ xoá các instance, VPC, IAM role và log group cùng metric filter. Chờ khoảng 3 phút đến khi stack biến mất khỏi danh sách **Active**.

![Xoá stack](/images/7-cleanup/7-delete-stack.png)

{{% notice info %}}
Metric không bị xoá. Các custom metric như `PythonAppErrorCount` và metric trong `CWAgent` sẽ ngừng nhận dữ liệu và tự hết hạn; bạn không bị tính phí cho metric không còn nhận dữ liệu mới.
{{% /notice %}}
