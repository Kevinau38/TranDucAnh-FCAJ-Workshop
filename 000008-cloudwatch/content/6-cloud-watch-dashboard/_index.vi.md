---
title : "CloudWatch Dashboards"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---


**Dashboard** gom các biểu đồ và alarm quan trọng vào một trang.

1. Ở **Alarms**, chọn **PythonAppErrorAlarm** (1), chọn **Actions** (2), rồi **Add to dashboard** (3).

![Add to dashboard](/images/6-dashboard/6-add-to-dashboard.png)

2. Chọn **Create new**, nhập `FCJ-CloudWatch-Workshop` (1) rồi chọn **Create** (2).

![Tạo dashboard](/images/6-dashboard/6-create-dashboard.png)

3. Dashboard mới được chọn (1). Giữ kiểu widget **Line** và chọn **Add to dashboard** (2).

![Add to dashboard](/images/6-dashboard/6-add-widget.png)

4. Thông báo xác nhận widget đã được thêm (1). Trong ảnh này alarm đang ở trạng thái **In alarm** (2) do các lỗi giả lập ở phần 5. Chọn **View dashboard**.

![Đã thêm widget](/images/6-dashboard/6-in-alarm.png)

5. Thêm widget thứ hai. Chọn **+** (Add widget) ở góc trên bên phải, giữ **Line** (1) rồi chọn **Next** (2).

![Add widget](/images/6-dashboard/6-widget-type.png)

6. Đổi tên biểu đồ thành `Memory used (%)` bằng biểu tượng bút chì (1). Tìm `mem_used_percent` (2), chọn **Graph search**, rồi chọn **Create widget** (3).

![Widget bộ nhớ](/images/6-dashboard/6-memory-widget.png)

7. Dashboard giờ có widget alarm (1), hiển thị màu đỏ khi alarm đang báo động, và widget bộ nhớ của ba instance (2). Chọn **Save** (3).

![Dashboard](/images/6-dashboard/6-dashboard.png)

{{% notice warning %}}
Thay đổi trên dashboard không được lưu tự động. Luôn chọn **Save** sau khi thêm hoặc di chuyển widget.
{{% /notice %}}
