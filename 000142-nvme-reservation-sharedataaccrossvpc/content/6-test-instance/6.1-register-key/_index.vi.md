---
title : "Đăng ký key cho instance test"
weight : 1
chapter : false
pre : " <b> 6.1 </b> "
---

1. Trên `test-postgres`, đăng ký key `0x2` và xem trạng thái reservation:

   ```bash
   sudo nvme resv-register /dev/nvme1n1 --nrkey=0x2 --rrega=0
   sudo nvme resv-report /dev/nvme1n1 --eds
   ```

   - (1) Key `0x2` đã được đăng ký.
   - (2) Reservation vẫn là loại `1` (Write Exclusive), và `regctl : 2`: có hai instance đã đăng ký.
   - (3) `rkey : 1` (`prod-postgres`) có `rcsts : 1`: vẫn đang giữ reservation.
   - (4) `rkey : 2` (`test-postgres`) có `rcsts : 0`: đã đăng ký nhưng không giữ reservation.

   ![Hai key đã đăng ký](/images/6-test-instance/6.1-register.png)

Cả hai instance đều thấy cùng một trạng thái, vì reservation được lưu trên volume chứ không phải trên instance.
