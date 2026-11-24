---
title : "Compare file transfer tools"
weight : 4
chapter : false
pre : " <b> 4.4 </b> "
---


In this step you copy the 2,000 sample files (2 GB) from the EBS volume to EFS with four common tools and compare the time. Before each copy, the page cache is dropped so that every tool reads the files from the EBS volume, not from memory.

1. Check the source data:

   ```bash
   du -csh /ebs/tutorial/data-1m/ && find /ebs/tutorial/data-1m/. -type f | wc -l
   ```

2. Copy the files with `rsync`, then with `cp`. Both are single-threaded:

   ```bash
   sudo sh -c 'sync && echo 3 > /proc/sys/vm/drop_caches' && time rsync -r /ebs/tutorial/data-1m/ /efs/tutorial/rsync/
   sudo sh -c 'sync && echo 3 > /proc/sys/vm/drop_caches' && time cp -r /ebs/tutorial/data-1m/* /efs/tutorial/cp/
   ```

3. Set the number of threads to 4 per vCPU (8 on a c7g.large), then copy the files with **fpsync**. fpsync uses `fpart` to split the file list into partitions and runs one `rsync` per partition, several at the same time. Run it twice, first with the default settings, then with `-f 250` (at most 250 files per partition):

   ```bash
   threads=$(($(nproc --all) * 4)); echo $threads; sudo sh -c 'sync && echo 3 > /proc/sys/vm/drop_caches' && time fpsync -n ${threads} /ebs/tutorial/data-1m/ /efs/tutorial/fpsync/
   sudo sh -c 'sync && echo 3 > /proc/sys/vm/drop_caches' && time fpsync -n ${threads} -f 250 /ebs/tutorial/data-1m/ /efs/tutorial/fpsync2/
   ```

4. Copy the files with `cp` and GNU `parallel`, 8 copies at the same time:

   ```bash
   sudo sh -c 'sync && echo 3 > /proc/sys/vm/drop_caches' && time find /ebs/tutorial/data-1m/. -type f | parallel --will-cite -j ${threads} cp {} /efs/tutorial/parallelcp
   ```

   - (1) `rsync`: about 1 minute 31 seconds.
   - (2) `cp`: about 1 minute 9 seconds.
   - (3) `fpsync` with default settings: about 1 minute 38 seconds, no better than `rsync`.
   - (4) `fpsync -f 250`: about 20 seconds.
   - (5) `cp` with GNU `parallel`: about 19 seconds.

   ![rsync, cp, fpsync and parallel cp](/images/4-efs/4.4-transfer-tools.png)

The first `fpsync` run is a useful surprise. By default `fpart` puts up to 2,000 files in one partition, so all 2,000 sample files land in a single partition and only one `rsync` runs: the 8 threads are never used. With `-f 250` the list is split into 8 partitions, 8 `rsync` processes run at the same time, and the copy is more than four times faster. Multi-threaded tools such as fpsync, `mcp` or GNU `parallel` take advantage of the distributed design of EFS, but only when the work is really split.

5. Optionally, open the [Amazon EFS console](https://console.aws.amazon.com/efs/), select `FCJ-StoragePerf-EFS` and review its (1) performance mode and throughput mode. The **Monitoring** tab shows the throughput and IOPS of the tests you just ran.

   ![EFS file system settings](/images/4-efs/4-efs-console.png)
