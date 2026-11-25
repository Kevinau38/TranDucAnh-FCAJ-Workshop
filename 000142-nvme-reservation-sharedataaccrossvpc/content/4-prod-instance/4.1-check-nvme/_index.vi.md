---
title : "Kiểm tra thiết bị NVMe"
weight : 1
chapter : false
pre : " <b> 4.1 </b> "
---

1. Trong EC2 console, chọn `prod-postgres`, chọn **Connect** → **Session Manager** → **Connect**. Một terminal mở ra trên trình duyệt với user `ssm-user`. Mở login shell:

   ```bash
   bash -l
   cd ~
   ```

2. Cài `nvme-cli` và PostgreSQL 18 từ repository của Amazon Linux 2023. Gói được tải qua S3 gateway endpoint:

   ```bash
   sudo dnf install -y -q nvme-cli postgresql18-server
   ```

3. Tìm volume dùng chung và kiểm tra khả năng reservation:

   ```bash
   lsblk -o NAME,SIZE,TYPE,MOUNTPOINTS
   sudo nvme list
   sudo nvme id-ns -H /dev/nvme1n1 | grep -A 8 rescap
   ```

   - (1) `nvme1n1` là volume 4 GiB, chưa format và chưa mount.
   - (2) Số serial (`SN`) của `/dev/nvme1n1` chính là volume ID bỏ dấu gạch, `vol0776038ead9621b7d`. Đây là cách đối chiếu thiết bị với volume EBS.
   - (3) `rescap` (reservation capabilities) bằng `0x7f`: volume hỗ trợ cả sáu loại reservation.

   ![Thiết bị NVMe và khả năng reservation](/images/4-prod-instance/4.1-nvme-list.png)

{{% notice note %}}
Ảnh chụp terminal trong bài được hiển thị với nền sáng cho dễ đọc. Terminal Session Manager của bạn sẽ có nền tối.
{{% /notice %}}
