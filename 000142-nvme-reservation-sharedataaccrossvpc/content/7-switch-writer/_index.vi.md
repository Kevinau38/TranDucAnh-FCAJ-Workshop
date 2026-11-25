---
title : "Chuyển quyền ghi"
weight : 7
chapter : false
pre : " <b> 7. </b> "
---

Giả sử bây giờ môi trường test cần ghi lên volume. Instance production nhả reservation, và instance test giành lấy.

1. Trên `prod-postgres`, nhả reservation:

   ```bash
   sudo nvme resv-release /dev/nvme1n1 --crkey=0x1 --rtype=1 --rrela=0
   sudo nvme resv-report /dev/nvme1n1 --eds
   ```

   - (1) Reservation đã được nhả (`--rrela=0` là release; `1` sẽ xoá toàn bộ đăng ký).
   - (2) `rtype : 0`: không instance nào giữ reservation.
   - (3) `rkey : 1` giờ có `rcsts : 0`. Cả hai key vẫn còn đăng ký.

   ![Đã nhả reservation](/images/7-switch-writer/7.1-release.png)

2. Trên `test-postgres`, giành reservation bằng key `0x2`, mount lại volume ở chế độ đọc ghi và ghi một file:

   ```bash
   sudo nvme resv-acquire /dev/nvme1n1 --crkey=0x2 --rtype=1 --racqa=0
   sudo umount /backup && sudo mount /dev/nvme1n1 /backup && echo "written by test-postgres at $(date -u)" | sudo tee /backup/handover.txt && ls -l /backup
   sudo nvme resv-report /dev/nvme1n1 --eds | grep -E 'rtype|rcsts|rkey'
   ```

   - (1) `test-postgres` giành được reservation Write Exclusive.
   - (2) Lệnh ghi thành công.
   - (3) `handover.txt` nằm cạnh file backup.
   - (4) `rkey : 2` giờ có `rcsts : 1`: `test-postgres` là instance duy nhất được ghi.

   ![Instance test giữ reservation](/images/7-switch-writer/7.2-acquire.png)

Các ứng dụng dùng chung volume cũng làm theo cách này. Một cluster manager như Pacemaker đăng ký key trên mỗi node và chỉ cho node đang hoạt động giữ reservation. Nếu node đó không còn phản hồi, một node khác sẽ **giành lại** (preempt) reservation (`resv-acquire --racqa=1 --prkey=<key cũ>`), và volume lập tức chặn mọi lệnh ghi muộn từ node cũ.
