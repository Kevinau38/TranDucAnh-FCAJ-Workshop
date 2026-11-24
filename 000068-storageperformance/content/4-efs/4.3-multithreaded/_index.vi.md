---
title : "Ghi đa luồng"
weight : 3
chapter : false
pre : " <b> 4.3 </b> "
---


Bước 4.2 cho thấy ghi block 1 MB và sync sau mỗi block rất chậm (khoảng 54 giây cho 2 GB). Trong bước này, bạn ghi cùng 2 GB đó, với cùng cấu hình, nhưng chia ra nhiều luồng.

1. Ghi 2 GB với 4 luồng (4 × 512 MB), rồi với 16 luồng (16 × 128 MB):

   ```bash
   time seq 0 3 | parallel --will-cite -j 4 dd if=/dev/zero of=/efs/tutorial/dd/2G-dd-$(date +%Y%m%d%H%M%S.%3N)-{} bs=1M count=512 oflag=sync status=none
   time seq 0 15 | parallel --will-cite -j 16 dd if=/dev/zero of=/efs/tutorial/dd/2G-dd-$(date +%Y%m%d%H%M%S.%3N)-{} bs=1M count=128 oflag=sync status=none
   du -sh /efs/tutorial/dd
   ```

   - (1) 4 luồng: khoảng 19 giây, nhanh gần gấp ba so với một luồng.
   - (2) 16 luồng: cũng khoảng 19 giây.

   ![Ghi 2 GB với 4 và 16 luồng](/images/4-efs/4.3-multithreaded.png)

Mỗi luồng vẫn phải chờ các lần sync của riêng nó, nhưng bốn luồng giữ bốn lần ghi chạy cùng lúc, đủ để đạt mức trần throughput của file system Bursting này (khoảng 110 MB/s). Thêm luồng cũng không vượt qua được mức trần đó. Trên file system dùng throughput Elastic hoặc Provisioned, các luồng thêm vào sẽ tiếp tục tăng throughput.
