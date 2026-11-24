---
title : "Chỉnh cấu hình sao lưu"
weight : 2
chapter : false
pre : " <b> 4.2. </b> "
---

#### Tổng quan

Bạn tăng thời gian lưu lên **3 ngày** và dời backup window sang **22:00 UTC** (05:00 giờ Việt Nam), lúc có ít truy cập. Việc đổi cấu hình sao lưu không khởi động lại cơ sở dữ liệu.

#### Đổi thời gian lưu và backup window

1. Chọn `rdspg-lab-db`, rồi **Modify**. Trong **Additional configuration** > **Backup**:
    - (1) **Backup retention period**: `3` ngày
    - (2) **Backup window**: **Choose a window**, **Start time** `22:00` UTC
    - (3) **Duration**: `0.5` giờ

    ![Backup settings](/images/4/4-2/1.png)

2. Chọn **Continue**:
    - (1) Kiểm tra thay đổi: **Backup retention period** `1` → `3`, và **Backup window** mới `22:00-22:30`.
    - (2) Chọn **Apply immediately**.
    - (3) Chọn **Modify DB instance**.

    ![Summary of modifications](/images/4/4-2/2.png)

{{% notice info %}}
Thời gian lưu automated backup từ 0 đến 35 ngày. Giá trị `0` tắt automated backup và point-in-time recovery, vì vậy đừng dùng cho production. Để giữ bản sao lưu lâu hơn 35 ngày, hãy dùng manual snapshot hoặc AWS Backup.
{{% /notice %}}

#### (Tùy chọn) AWS CLI

{{%expand "Đổi cấu hình sao lưu bằng AWS CLI" %}}
```bash
aws rds modify-db-instance \
    --db-instance-identifier rdspg-lab-db \
    --backup-retention-period 3 \
    --preferred-backup-window 22:00-22:30 \
    --apply-immediately
```
{{% /expand %}}
