---
title : "Optimize throughput"
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---


In this step you upload one 5 GB file with more and more parallel requests, then upload the same amount of data as five separate files.

#### Upload one large file

1. Configure the AWS CLI to send one request at a time, and to upload files larger than 64 MB in parts of 16 MB. Then create a 5 GB test file:

   ```bash
   aws configure set default.s3.max_concurrent_requests 1 && aws configure set default.s3.multipart_threshold 64MB && aws configure set default.s3.multipart_chunksize 16MB && cat ~/.aws/config
   dd if=/dev/urandom of=5GB.file bs=1 count=0 seek=5G && ls -lh 5GB.file
   ```

   - (1) The three settings are written to `~/.aws/config`.
   - (2) `dd` with `count=0 seek=5G` creates a 5 GB file instantly (a sparse file), so you do not wait for random data to be written.

   ![AWS CLI S3 settings](/images/3-s3/3.1-config.png)

2. Upload the file four times, with 1, 2, 10 and 20 concurrent requests:

   ```bash
   time aws s3 cp 5GB.file s3://${bucket}/upload1.test --no-progress
   aws configure set default.s3.max_concurrent_requests 2 && time aws s3 cp 5GB.file s3://${bucket}/upload2.test --no-progress
   aws configure set default.s3.max_concurrent_requests 10 && time aws s3 cp 5GB.file s3://${bucket}/upload3.test --no-progress
   aws configure set default.s3.max_concurrent_requests 20 && time aws s3 cp 5GB.file s3://${bucket}/upload4.test --no-progress
   ```

   In the screenshot, the upload took about 1 minute with 1 request, 31 seconds with 2, and about 9 seconds with both 10 and 20 requests.

   ![Upload with 1, 2, 10 and 20 threads](/images/3-s3/3.1-uploads.png)

The 5 GB file is split into 320 parts of 16 MB. With one request, the parts are sent one after the other. With more requests, several parts travel at the same time, so the upload is much faster. From 10 to 20 requests there is no more gain: the AWS CLI itself (and the CPU of a 2 vCPU instance) has become the limit. Tools written for high throughput, such as the AWS SDKs with the CRT transfer client, can use hundreds of connections and go further.

#### Upload several files at the same time

3. Create a 1 GB file and upload it five times in parallel with GNU `parallel`. Five AWS CLI processes run at the same time, each with 20 concurrent requests:

   ```bash
   dd if=/dev/urandom of=1GB.file bs=1 count=0 seek=1G && ls -lh 1GB.file
   time seq 1 5 | parallel --will-cite -j 5 aws s3 cp 1GB.file s3://${bucket}/parallel/object{}.test --no-progress
   ```

   - (1) `-j 5` runs five uploads at the same time.
   - (2) The same 5 GB of data is uploaded in about 7 seconds.

   ![Upload five 1 GB files in parallel](/images/3-s3/3.1-parallel.png)

Parallelism costs requests. Each part is a separate PUT request, billed per 1,000 requests: a 5 GB file in 16 MB parts is 320 PUT requests instead of 1. You pay a little more in requests to go faster.
