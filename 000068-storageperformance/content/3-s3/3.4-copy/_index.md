---
title : "Optimize copy operations"
weight : 4
chapter : false
pre : " <b> 3.4 </b> "
---


To copy an object to another location in S3, you do not need to download it first. Amazon S3 can copy the data inside the service (**PUT Copy**, the `CopyObject` and `UploadPartCopy` APIs), so the instance only sends API calls and the data never leaves S3.

1. Copy the 5 GB object `upload1.test` to the `copy/` prefix in three ways:

   ```bash
   aws configure set default.s3.max_concurrent_requests 10
   time (aws s3 cp s3://$bucket/upload1.test 5GB.file --no-progress; aws s3 cp 5GB.file s3://$bucket/copy/5GB.file --no-progress)
   time aws s3api copy-object --copy-source $bucket/upload1.test --bucket $bucket --key copy/5GB-2.file --query CopyObjectResult.ETag --output text
   time aws s3 cp s3://$bucket/upload1.test s3://$bucket/copy/5GB-3.file --no-progress
   ```

   - (1) Download to the instance, then upload again: about 1 minute 20 seconds. 10 GB travel between the instance and S3.
   - (2) `aws s3api copy-object`: about 1 minute 17 seconds. The copy happens inside S3, but in one single request.
   - (3) `aws s3 cp` from S3 to S3: about 10 seconds. The AWS CLI splits the copy into parts and copies up to 10 parts at the same time with `UploadPartCopy`.

   ![Three ways to copy a 5 GB object](/images/3-s3/3.4-copy.png)
2. Open the [Amazon S3 console](https://console.aws.amazon.com/s3/) and select the lab bucket. (1) The prefixes `copy/`, `parallel/`, `run1/` to `run3/`, `sync1/` and `sync2/` hold the objects you uploaded in this part, next to the four `upload*.test` objects.

   ![Objects in the lab bucket](/images/3-s3/3.4-bucket.png)

Methods (2) and (3) both copy inside S3, so no data goes through the instance and no data transfer is charged. Method (3) is the fastest because it combines server-side copy with parallel parts. A single `CopyObject` request also has a limit of 5 GB, while a multipart copy works for objects up to 5 TB.

{{% notice note %}}
For multipart copies the AWS CLI also copies the object tags, so the role needs `s3:GetObjectTagging` and `s3:PutObjectTagging` on the bucket. The lab role already has them.
{{% /notice %}}
