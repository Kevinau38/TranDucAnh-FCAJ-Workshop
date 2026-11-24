---
title : "Xem metric"
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---


1. Mở console **CloudWatch**. Ở thanh điều hướng, mở **Metrics** (1) và chọn **Classic metrics** (2). Trong **Custom namespaces** bạn thấy `CWAgent` do CloudWatch agent tạo ra (3). Namespace `FCJ/CloudWatchWorkshop` sẽ xuất hiện sau phần 4.3.

![Classic metrics](/images/3-metrics/3.1-classic-metrics.png)

2. Chọn khoảng thời gian **1h**. Ở ô tìm kiếm nhập `EC2` rồi nhấn **Enter** (1), sau đó chọn **EC2 > Per-Instance Metrics** (2).

![EC2 per-instance metrics](/images/3-metrics/3.1-search-ec2.png)

3. Ở ô tìm kiếm nhập `CPUUtilization` rồi nhấn **Enter**. Sắp xếp theo **Instance name** và chọn **FCJ-CloudWatch-Instance-A**, **FCJ-CloudWatch-Instance-B** (1). Biểu đồ hiển thị hai đường (2). Cả hai instance lên xuống theo chu kỳ, A tải nặng hơn B.

![CPU của hai instance](/images/3-metrics/3.1-cpu-two-instances.png)

{{% notice info %}}
Tài khoản của bạn có thể có instance khác. Chỉ chọn những dòng có tên bắt đầu bằng `FCJ-CloudWatch-`.
{{% /notice %}}

4. Bỏ chọn Instance B. Xoá bộ lọc `CPUUtilization`, tìm `EBSWriteBytes` và chọn dòng của **Instance-A** (1). Biểu đồ giờ hiển thị CPU và lượng ghi đĩa của cùng một instance (2). Hai đường biến động cùng nhau vì workload ghi đĩa ngay sau giai đoạn dùng CPU.

![CPU và EBS write bytes](/images/3-metrics/3.1-cpu-and-ebs.png)

5. Hai metric có đơn vị khác nhau (phần trăm và byte) nên đường CPU gần như phẳng. Mở tab **Graphed metrics**, ở dòng `EBSWriteBytes` chọn mũi tên phải trong cột **Y axis** (1). Metric được chuyển sang trục Y thứ hai bên phải (2) và cả hai đường đều dễ đọc.

![Trục Y bên phải](/images/3-metrics/3.1-right-axis.png)
