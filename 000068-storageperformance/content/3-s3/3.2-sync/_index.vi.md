---
title : "Lệnh sync"
weight : 2
chapter : false
pre : " <b> 3.2 </b> "
---


Lệnh `aws s3 sync` copy các file còn thiếu hoặc đã thay đổi giữa một thư mục local và một bucket (hoặc giữa hai bucket). Với tuỳ chọn `--delete`, lệnh cũng xoá ở đích những file không còn ở nguồn.

1. Sync 2.000 file mẫu (khoảng 2 GB) lên bucket với 1 request, rồi với 10 request:

   ```bash
   aws configure set default.s3.max_concurrent_requests 1 && time aws s3 sync /ebs/tutorial/data-1m/ s3://${bucket}/sync1/ --only-show-errors
   aws configure set default.s3.max_concurrent_requests 10 && time aws s3 sync /ebs/tutorial/data-1m/ s3://${bucket}/sync2/ --only-show-errors
   ```

   - (1) Với 1 request, lệnh sync mất khoảng 1 phút 40 giây.
   - (2) Với 10 request, chỉ mất khoảng 16 giây.

   ![aws s3 sync với 1 và 10 luồng](/images/3-s3/3.2-sync.png)

Các file này chỉ 1 MB, nhỏ hơn ngưỡng multipart, nên mỗi file là một request PUT. CLI chạy nhanh hơn vì upload tới 10 file cùng lúc.
