---
title : "Multi-threaded writes"
weight : 3
chapter : false
pre : " <b> 4.3 </b> "
---


Step 4.2 showed that writing 1 MB blocks with a sync after every block is slow (about 54 seconds for 2 GB). In this step you write the same 2 GB, with the same settings, but split over several threads.

1. Write 2 GB with 4 threads (4 × 512 MB), then with 16 threads (16 × 128 MB):

   ```bash
   time seq 0 3 | parallel --will-cite -j 4 dd if=/dev/zero of=/efs/tutorial/dd/2G-dd-$(date +%Y%m%d%H%M%S.%3N)-{} bs=1M count=512 oflag=sync status=none
   time seq 0 15 | parallel --will-cite -j 16 dd if=/dev/zero of=/efs/tutorial/dd/2G-dd-$(date +%Y%m%d%H%M%S.%3N)-{} bs=1M count=128 oflag=sync status=none
   du -sh /efs/tutorial/dd
   ```

   - (1) 4 threads: about 19 seconds, almost three times faster than one thread.
   - (2) 16 threads: also about 19 seconds.

   ![Write 2 GB with 4 and 16 threads](/images/4-efs/4.3-multithreaded.png)

Each thread still waits for its own syncs, but four threads keep four writes in flight at the same time, which is enough to reach the throughput ceiling of this Bursting file system (about 110 MB/s). Adding more threads cannot go past that ceiling. On a file system with Elastic or Provisioned throughput, the extra threads would keep adding throughput.
