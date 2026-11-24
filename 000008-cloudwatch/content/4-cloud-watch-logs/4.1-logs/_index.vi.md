---
title : "Log group và log stream"
weight : 1
chapter : false
pre : " <b> 4.1 </b> "
---


1. Ở thanh điều hướng CloudWatch, mở **Logs** và chọn **Log Management** (1). Lọc theo `/fcj/cloudwatch-workshop` (2) rồi mở log group **/fcj/cloudwatch-workshop/var/log/messages** (3).

![Log group](/images/4-logs/4.1-log-groups.png)

2. Tab **Log streams** liệt kê mỗi instance một stream, đặt tên theo instance ID (1). Mở một stream bất kỳ.

![Log stream](/images/4-logs/4.1-log-streams.png)

3. Bạn thấy log hệ thống của instance: `systemd`, `amazon-ssm-agent` và các dịch vụ khác (1). Có thể dùng ô lọc để tìm, ví dụ `ssm`.

![Log event](/images/4-logs/4.1-log-events.png)

4. Quay lại log group. Mặc định log không bao giờ hết hạn, lâu dài sẽ tốn chi phí. Chọn **Actions** (1), rồi **Edit retention setting** (2).

![Edit retention setting](/images/4-logs/4.1-actions-retention.png)

5. Đặt **Expire events after** là **1 week (7 days)** (1) rồi chọn **Save** (2).

![Retention setting](/images/4-logs/4.1-retention-dialog.png)

6. Thông báo xác nhận thay đổi xuất hiện (1).

![Đã cập nhật retention](/images/4-logs/4.1-retention-saved.png)
