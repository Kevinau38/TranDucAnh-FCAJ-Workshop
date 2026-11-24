---
title : "I/O size and sync frequency"
weight : 2
chapter : false
pre : " <b> 4.2 </b> "
---


In this step you write a 2 GB file to EFS four times, changing the block size (`bs`) and how often the data is synced to EFS:

- `conv=fsync`: sync once, at the end of the file.
- `oflag=sync`: sync after every block.

1. Run the four commands:

   ```bash
   time dd if=/dev/zero of=/efs/tutorial/dd/2G-dd-$(date +%Y%m%d%H%M%S.%3N) bs=1M count=2048 conv=fsync
   time dd if=/dev/zero of=/efs/tutorial/dd/2G-dd-$(date +%Y%m%d%H%M%S.%3N) bs=16M count=128 conv=fsync
   time dd if=/dev/zero of=/efs/tutorial/dd/2G-dd-$(date +%Y%m%d%H%M%S.%3N) bs=1M count=2048 oflag=sync
   time dd if=/dev/zero of=/efs/tutorial/dd/2G-dd-$(date +%Y%m%d%H%M%S.%3N) bs=16M count=128 oflag=sync
   ```

   - (1) 1 MB blocks, sync once: about 19 seconds, 113 MB/s.
   - (2) 16 MB blocks, sync once: about 19 seconds, 112 MB/s.
   - (3) 1 MB blocks, sync after every block: about 54 seconds, 40 MB/s.
   - (4) 16 MB blocks, sync after every block: about 19 seconds, 115 MB/s.

   ![dd with different block sizes and sync settings](/images/4-efs/4.2-sync-frequency.png)

When the data is synced only at the end, the NFS client groups the writes, and the file system reaches its throughput ceiling whatever the block size. Syncing after every small block forces one round trip to EFS per block and cuts the throughput by about three times. If your application must sync often, use larger I/O sizes, as in test (4).
