---
title : "Dọn dẹp tài nguyên"
weight : 10
chapter : false
pre : " <b> 10. </b> "
---


#### Dọn dẹp tài nguyên

Xóa tài nguyên theo thứ tự sau để không còn gì phát sinh chi phí.

#### IAM Access Analyzer

1. IAM console → **Access analyzer** → **Analyzer settings**, chọn `FCJ-S3-Analyzer` → **Delete**.

#### AWS Config (chỉ khi bạn đã thiết lập ở phần 8)

2. AWS Config → **Rules**, chọn `s3-bucket-public-read-prohibited` → **Actions** → **Delete rule**.
3. AWS Config → **Settings** → **Edit**, bỏ chọn **Enable recording** và lưu. Bạn cũng có thể chạy các lệnh sau trong **CloudShell**:

```bash
aws configservice stop-configuration-recorder --configuration-recorder-name default
aws configservice delete-delivery-channel --delivery-channel-name default
aws configservice delete-configuration-recorder --configuration-recorder-name default
```

4. Empty và xóa bucket delivery của Config `fcj-config-lab-<suffix>`.

#### VPC endpoint và bucket

5. VPC console → **Endpoints**, chọn `S3-Endpoint` → **Actions** → **Delete VPC endpoints**.
6. S3 console → mở bucket lab → **Permissions**, xóa bucket policy nếu vẫn còn.
7. Chọn bucket lab → **Empty**, nhập `permanently delete` và xác nhận. (CloudFormation stack không xóa được bucket còn chứa object.)

#### CloudFormation stack

8. Đóng tab Session Manager (**Terminate session**).
9. CloudFormation console → chọn `FCJ-S3-LAB` → **Delete**. Stack sẽ xóa EC2 instance, IAM role và instance profile, bucket, security group và VPC.
10. Chờ trạng thái **DELETE_COMPLETE**, rồi kiểm tra `SID-vpc` và bucket `sid-security-*` không còn tồn tại.
