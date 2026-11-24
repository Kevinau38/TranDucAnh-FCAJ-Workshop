---
title : "Giới thiệu"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---


#### Amazon CloudWatch

CloudWatch là dịch vụ giám sát và quan sát hệ thống của AWS. Các thành phần chính bạn dùng trong bài lab:

| Tính năng | Chức năng |
|---|---|
| **Metrics** | Chuỗi số theo thời gian, ví dụ `CPUUtilization` của một EC2 instance. Metric thuộc một **namespace** (ví dụ `AWS/EC2`) và được xác định bởi các **dimension** (ví dụ `InstanceId`). |
| **CloudWatch agent** | Chạy trên máy chủ, gửi thêm metric (bộ nhớ, ổ đĩa, theo từng process) và file log lên CloudWatch. Metric của agent nằm trong namespace `CWAgent`. |
| **Logs** | Log event được lưu trong **log group** và **log stream**. **Logs Insights** truy vấn log bằng một ngôn ngữ truy vấn đơn giản. **Metric filter** chuyển các dòng log khớp mẫu thành metric. |
| **Alarms** | Theo dõi một metric hoặc expression và đổi trạng thái (`OK`, `ALARM`, `INSUFFICIENT_DATA`) khi vượt ngưỡng. Alarm có thể gửi thông báo tới một **Amazon SNS** topic. |
| **Dashboards** | Trang tùy chỉnh chứa biểu đồ và widget alarm. |

#### Môi trường lab

CloudFormation stack tạo ba instance **FCJ-CloudWatch-Instance-A**, **B** và **C**. Mỗi instance:

- chạy một workload nhỏ dùng CPU và ghi đĩa theo chu kỳ (Instance A nặng nhất, C nhẹ nhất), để metric không bị phẳng;
- chạy CloudWatch agent, gửi `mem_used_percent`, `disk_used_percent` và metric theo process của chính agent vào namespace `CWAgent`, đồng thời đẩy `/var/log/messages` lên log group `/fcj/cloudwatch-workshop/var/log/messages`;
- có sẵn script `/opt/fcj/logger.py`, một ứng dụng web giả lập ghi các dòng `INFO`, `WARNING` và `ERROR` vào syslog. Bạn sẽ chạy script này ở phần 4.2.

Các instance không mở inbound rule và không có SSH key. Bạn mở shell bằng **Session Manager**.
