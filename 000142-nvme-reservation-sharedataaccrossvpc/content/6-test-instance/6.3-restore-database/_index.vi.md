---
title : "Restore database"
weight : 3
chapter : false
pre : " <b> 6.3 </b> "
---

1. Mount volume ở chế độ **chỉ đọc**, tạo database `restore`, nạp bản dump vào và truy vấn bảng:

   ```bash
   sudo mkdir -p /backup && sudo mount -o ro,norecovery /dev/nvme1n1 /backup && ls -l /backup
   sudo postgresql-setup --initdb && sudo systemctl enable --now postgresql && sudo -u postgres createdb restore
   sudo -u postgres psql -q -d restore -f /backup/test_erp.sql && sudo -u postgres psql -d restore -c "SELECT * FROM clients;"
   ```

   - (1) Mount chỉ đọc thấy file `test_erp.sql` do `prod-postgres` ở VPC khác ghi.
   - (2) Bảng `clients` trong database `restore` có đúng ba dòng như ở production.

   ![Database đã restore](/images/6-test-instance/6.3-restore.png)

`ro` mount file system ở chế độ chỉ đọc, còn `norecovery` báo XFS không replay log, vì việc đó cần ghi. Dữ liệu đã đi từ `prod-vpc` sang `test-vpc` mà không có bất kỳ kết nối mạng nào giữa hai VPC.
