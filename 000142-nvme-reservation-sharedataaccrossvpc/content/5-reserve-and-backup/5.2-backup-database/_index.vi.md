---
title : "Backup database ra volume dùng chung"
weight : 2
chapter : false
pre : " <b> 5.2 </b> "
---

1. Tạo file system XFS trên volume dùng chung và tạo thư mục mount. Chỉ làm một lần, trên instance đang giữ reservation:

   ```bash
   sudo mkfs.xfs -q /dev/nvme1n1
   sudo mkdir -p /backup
   ```

2. Mount volume, ghi bản dump của `test_erp` bằng `pg_dump`, rồi unmount:

   ```bash
   sudo mount /dev/nvme1n1 /backup && df -h /backup
   sudo chown postgres:postgres /backup
   sudo -u postgres pg_dump -d test_erp -f /backup/test_erp.sql && ls -l /backup && grep -A 4 '^COPY' /backup/test_erp.sql
   sudo umount /backup && echo UNMOUNTED
   ```

   - (1) Volume 4 GiB đã được mount vào `/backup`.
   - (2) File dump `test_erp.sql` đã được ghi lên volume.
   - (3) File dump chứa ba dòng của bảng `clients`.
   - (4) Volume đã được unmount.

   ![Backup trên volume dùng chung](/images/5-reserve-and-backup/5.2-backup.png)

Bước unmount rất quan trọng. XFS giữ dữ liệu và metadata trong page cache của instance, instance thứ hai không thấy được cho tới khi chúng được ghi xuống đĩa. `umount` đẩy toàn bộ xuống volume, nhờ vậy instance test đọc được một file system đầy đủ và nhất quán. Sau khi unmount, `prod-postgres` vẫn giữ reservation.
