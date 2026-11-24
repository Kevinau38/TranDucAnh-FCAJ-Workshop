---
title : "Kích thước I/O và tần suất sync"
weight : 2
chapter : false
pre : " <b> 4.2 </b> "
---


Trong bước này, bạn ghi một file 2 GB lên EFS bốn lần, thay đổi kích thước block (`bs`) và tần suất đồng bộ (sync) dữ liệu xuống EFS:

- `conv=fsync`: sync một lần, ở cuối file.
- `oflag=sync`: sync sau mỗi block.

1. Chạy bốn lệnh sau:

   ```bash
   time dd if=/dev/zero of=/efs/tutorial/dd/2G-dd-$(date +%Y%m%d%H%M%S.%3N) bs=1M count=2048 conv=fsync
   time dd if=/dev/zero of=/efs/tutorial/dd/2G-dd-$(date +%Y%m%d%H%M%S.%3N) bs=16M count=128 conv=fsync
   time dd if=/dev/zero of=/efs/tutorial/dd/2G-dd-$(date +%Y%m%d%H%M%S.%3N) bs=1M count=2048 oflag=sync
   time dd if=/dev/zero of=/efs/tutorial/dd/2G-dd-$(date +%Y%m%d%H%M%S.%3N) bs=16M count=128 oflag=sync
   ```

   - (1) Block 1 MB, sync một lần: khoảng 19 giây, 113 MB/s.
   - (2) Block 16 MB, sync một lần: khoảng 19 giây, 112 MB/s.
   - (3) Block 1 MB, sync sau mỗi block: khoảng 55 giây, 39 MB/s.
   - (4) Block 16 MB, sync sau mỗi block: khoảng 19 giây, 115 MB/s.

   ![dd với các kích thước block và cách sync khác nhau](/images/4-efs/4.2-sync-frequency.png)

Khi dữ liệu chỉ được sync ở cuối, NFS client gom các lần ghi lại với nhau, và file system đạt mức trần throughput bất kể kích thước block. Sync sau mỗi block nhỏ buộc mỗi block phải chờ một lượt đi về tới EFS, làm throughput giảm khoảng ba lần. Nếu ứng dụng của bạn bắt buộc phải sync thường xuyên, hãy dùng kích thước I/O lớn hơn, như bài đo (4).
