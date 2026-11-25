---
title : "Giành reservation"
weight : 1
chapter : false
pre : " <b> 5.1 </b> "
---

1. Trên `prod-postgres`, đăng ký key `0x1`, dùng key đó giành reservation Write Exclusive, rồi xem trạng thái reservation:

   ```bash
   sudo nvme resv-register /dev/nvme1n1 --nrkey=0x1 --rrega=0
   sudo nvme resv-acquire /dev/nvme1n1 --crkey=0x1 --rtype=1 --racqa=0
   sudo nvme resv-report /dev/nvme1n1 --eds
   ```

   | Tuỳ chọn | Ý nghĩa |
   |---|---|
   | `--nrkey=0x1` | Key reservation mới của instance |
   | `--rrega=0` | Register action `0`: đăng ký key |
   | `--crkey=0x1` | Key hiện tại, dùng để xác nhận ai gửi lệnh |
   | `--rtype=1` | Loại reservation `1`: Write Exclusive |
   | `--racqa=0` | Acquire action `0`: giành reservation (`1` là giành lại từ instance đang giữ) |
   | `--eds` | Báo cáo dạng extended data structure, hiện đủ key 64 bit |

   - (1) Key đã được đăng ký.
   - (2) Đã giành được reservation.
   - (3) `rtype : 1` (Write Exclusive) và `regctl : 1` (một instance đã đăng ký).
   - (4) Bên đăng ký có `rkey : 1` có `rcsts : 1`, nghĩa là đang giữ reservation.

   ![Reservation Write Exclusive](/images/5-reserve-and-backup/5.1-acquire.png)
