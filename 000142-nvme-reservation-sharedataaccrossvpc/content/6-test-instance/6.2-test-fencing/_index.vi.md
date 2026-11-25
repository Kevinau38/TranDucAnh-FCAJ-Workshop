---
title : "Kiểm chứng lệnh ghi bị chặn"
weight : 2
chapter : false
pre : " <b> 6.2 </b> "
---

1. Thử ghi trực tiếp một block 4 KiB vào thiết bị, ở gần cuối volume, rồi đọc 1 MiB từ đầu volume:

   ```bash
   sudo dd if=/dev/zero of=/dev/nvme1n1 bs=4k count=1 seek=1048000 oflag=direct
   sudo dd if=/dev/nvme1n1 of=/dev/null bs=4k count=256 iflag=direct
   ```

   - (1) Lệnh ghi thất bại với lỗi `Invalid exchange`. Đây là cách Linux báo trạng thái NVMe **Reservation Conflict**: volume từ chối lệnh ghi vì `prod-postgres` đang giữ reservation Write Exclusive.
   - (2) Lệnh đọc thành công. Reservation Write Exclusive vẫn cho các instance khác đọc.

   ![Ghi bị chặn, đọc được](/images/6-test-instance/6.2-write-blocked.png)

{{% notice warning %}}
Đừng bao giờ ghi thẳng vào thiết bị đang chứa file system. Lệnh thử ghi nhắm vào một block gần cuối volume 4 GiB, vùng mà file system XFS nhỏ này không dùng, và dù sao lệnh cũng bị từ chối. Nếu không có reservation, cùng lệnh đó sẽ ghi thành công.
{{% /notice %}}
