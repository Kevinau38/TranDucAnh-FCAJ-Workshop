---
title : "Dọn dẹp tài nguyên"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---


1. Đóng terminal Session Manager bằng nút **Terminate session**.

2. Làm rỗng bucket của bài lab. CloudFormation không xoá được bucket còn chứa object. Mở [Amazon S3 console](https://console.aws.amazon.com/s3/), chọn bucket `fcj-storageperf-labbucket-...` rồi chọn **Empty**. (1) Nhập `permanently delete`, rồi (2) chọn **Empty**.

   ![Empty bucket](/images/5-cleanup/5-empty-confirm.png)

3. Chờ tới khi (1) bucket đã được làm rỗng. (2) Toàn bộ object thử nghiệm đã bị xoá.

   ![Bucket đã rỗng](/images/5-cleanup/5-empty-done.png)

4. Mở [AWS CloudFormation console](https://console.aws.amazon.com/cloudformation/), chọn stack `FCJ-StoragePerf` rồi chọn **Delete**. (1) Nhập `FCJ-StoragePerf`, rồi (2) chọn **Delete stack**.

   ![Delete stack](/images/5-cleanup/5-delete-stack.png)

5. Chờ tới khi stack biến mất khỏi danh sách (lọc **Deleted** để thấy trạng thái **DELETE_COMPLETE**), khoảng 5 phút. CloudFormation xoá instance cùng các ổ EBS, EFS file system, bucket đã rỗng, IAM role, các security group và VPC.

   ![DELETE_COMPLETE](/images/5-cleanup/5-delete-complete.png)

6. Trên máy của bạn, bạn có thể xoá file `fcj-storage-performance.yaml`.
