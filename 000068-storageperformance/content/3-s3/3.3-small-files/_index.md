---
title : "Optimize small file operations"
weight : 3
chapter : false
pre : " <b> 3.3 </b> "
---


For small objects, the time is spent on requests, not on data. The way to move many small objects faster is to have more requests in flight at the same time, so that you get more **transactions per second (TPS)**.

1. Create 500 files of 1 KB in the folder `small/`:

   ```bash
   seq 1 500 > object_ids && dd if=/dev/urandom of=1KB.file bs=1 count=0 seek=1K 2>/dev/null
   mkdir -p small && for i in $(seq 1 500); do cp 1KB.file small/object-$i; done; ls small | wc -l
   ```

2. Upload the folder three times with 1, 10 and 50 concurrent requests:

   ```bash
   aws configure set default.s3.max_concurrent_requests 1 && time aws s3 cp small/ s3://${bucket}/run1/ --recursive --only-show-errors
   aws configure set default.s3.max_concurrent_requests 10 && time aws s3 cp small/ s3://${bucket}/run2/ --recursive --only-show-errors
   aws configure set default.s3.max_concurrent_requests 50 && time aws s3 cp small/ s3://${bucket}/run3/ --recursive --only-show-errors
   ```

   - (1) With 1 request: about 17 seconds, around 30 objects per second.
   - (2) With 10 requests: about 2.1 seconds, around 235 objects per second.
   - (3) With 50 requests: no faster than with 10.

   ![Upload 500 small files with 1, 10 and 50 threads](/images/3-s3/3.3-small-files.png)

From 10 to 50 requests the time stays the same: the bottleneck is now the single CLI process and the CPU of the instance, not Amazon S3. To go further you would spread the work over more processes or more instances.

{{% notice note %}}
The original version of this step started one `aws s3 cp` command per object with GNU `parallel` (`parallel -a object_ids -j 5 aws s3 cp ...`). On a 2 vCPU instance that takes about 2 minutes for 500 objects whatever the number of jobs, because starting 500 AWS CLI processes uses all the CPU. One process with many concurrent requests is far more efficient.
{{% /notice %}}
