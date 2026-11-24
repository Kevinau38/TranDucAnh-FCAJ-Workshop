---
title : "Yêu cầu mã hóa SSE-S3"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---


#### Yêu cầu mã hóa SSE-S3

Hiện nay S3 mặc định mã hóa mọi object mới bằng SSE-S3. Tuy vậy, một số tổ chức vẫn yêu cầu client phải **chủ động** yêu cầu mã hóa, để request upload không có header mã hóa bị từ chối. Bạn có thể áp dụng điều này bằng bucket policy với condition key `s3:x-amz-server-side-encryption`.

1. Mở lại **Permissions** → **Bucket policy** → **Edit**.
2. Thay policy bằng nội dung sau và chọn **Save changes**:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "RequireSSES3",
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::<your-bucket-name>/*",
            "Condition": {
                "StringNotEquals": {
                    "s3:x-amz-server-side-encryption": "AES256"
                }
            }
        }
    ]
}
```

![Policy yêu cầu SSE-S3](/images/4-sse/4-policy.png)

3. Trong terminal, tạo file thử nghiệm và upload lần lượt không có và có header mã hóa:

```bash
echo "123456789abcdefg" > textfile
aws s3api put-object --key text01 --body textfile --bucket $bucket
aws s3api put-object --key text01 --body textfile --server-side-encryption AES256 --bucket $bucket
```

Lần upload đầu bị lỗi **AccessDenied ... with an explicit deny in a resource-based policy**; lần thứ hai thành công và trả về `"ServerSideEncryption": "AES256"`.

![Upload không có SSE bị từ chối](/images/4-sse/4-test.png)
