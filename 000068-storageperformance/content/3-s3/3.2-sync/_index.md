---
title : "Sync command"
weight : 2
chapter : false
pre : " <b> 3.2 </b> "
---


`aws s3 sync` copies the files that are missing or have changed between a local directory and a bucket (or between two buckets). With `--delete` it also removes from the destination the files that are not in the source.

1. Sync the 2,000 sample files (about 2 GB) to the bucket with 1 request, then with 10 requests:

   ```bash
   aws configure set default.s3.max_concurrent_requests 1 && time aws s3 sync /ebs/tutorial/data-1m/ s3://${bucket}/sync1/ --only-show-errors
   aws configure set default.s3.max_concurrent_requests 10 && time aws s3 sync /ebs/tutorial/data-1m/ s3://${bucket}/sync2/ --only-show-errors
   ```

   - (1) With 1 request, the sync takes about 1 minute 40 seconds.
   - (2) With 10 requests, it takes about 16 seconds.

   ![aws s3 sync with 1 and 10 threads](/images/3-s3/3.2-sync.png)

These files are only 1 MB, smaller than the multipart threshold, so each file is one PUT request. The CLI goes faster because it uploads up to 10 files at the same time.
