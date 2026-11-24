---
title : "Clean up resources"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---


1. Close the Session Manager terminal with **Terminate session**.

2. Empty the lab bucket. CloudFormation cannot delete a bucket that still contains objects. Open the [Amazon S3 console](https://console.aws.amazon.com/s3/), select the bucket `fcj-storageperf-labbucket-...` and choose **Empty**. (1) Type `permanently delete`, then (2) choose **Empty**.

   ![Empty bucket](/images/5-cleanup/5-empty-confirm.png)

3. Wait until (1) the bucket is emptied. (2) All the test objects are deleted.

   ![Bucket emptied](/images/5-cleanup/5-empty-done.png)

4. Open the [AWS CloudFormation console](https://console.aws.amazon.com/cloudformation/), select the stack `FCJ-StoragePerf` and choose **Delete**. (1) Type `FCJ-StoragePerf`, then (2) choose **Delete stack**.

   ![Delete stack](/images/5-cleanup/5-delete-stack.png)

5. Wait until the stack disappears from the list (filter **Deleted** to see **DELETE_COMPLETE**), about 5 minutes. CloudFormation deletes the instance and its EBS volumes, the EFS file system, the empty bucket, the IAM role, the security groups and the VPC.

   ![DELETE_COMPLETE](/images/5-cleanup/5-delete-complete.png)

6. On your own machine, you can delete `fcj-storage-performance.yaml`.
