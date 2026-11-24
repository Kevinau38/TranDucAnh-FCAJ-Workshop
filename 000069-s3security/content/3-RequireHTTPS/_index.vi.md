---
title : "Yêu cầu HTTPS"
weight : 3
chapter : false
pre : " <b> 3. </b> "
---


#### Yêu cầu HTTPS

Amazon S3 chấp nhận request qua cả HTTP và HTTPS. Để dữ liệu luôn được mã hóa khi truyền, hãy thêm bucket policy từ chối mọi request có `aws:SecureTransport` bằng `false`.

1. Trong S3 console, mở bucket của bạn và chọn tab **Permissions**.
2. Tại **Bucket policy**, chọn **Edit**.

![Quyền của bucket](/images/3-https/3-permissions.png)

3. Dán policy sau và chọn **Save changes**:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "RequireHTTPS",
            "Action": "s3:*",
            "Effect": "Deny",
            "Principal": "*",
            "Resource": "arn:aws:s3:::<your-bucket-name>/*",
            "Condition": {
                "Bool": {
                    "aws:SecureTransport": "false"
                }
            }
        }
    ]
}
```

![Policy yêu cầu HTTPS](/images/3-https/3-policy.png)

4. Trong terminal Session Manager, đọc metadata của object qua **HTTP** rồi qua **HTTPS**:

```bash
aws s3api head-object --key app1/file1 --endpoint-url http://s3.ap-northeast-1.amazonaws.com --bucket $bucket
aws s3api head-object --key app1/file1 --endpoint-url https://s3.ap-northeast-1.amazonaws.com --bucket $bucket
```

Request HTTP bị từ chối với lỗi **403 Forbidden**; request HTTPS thành công.

![HTTP bị từ chối, HTTPS được phép](/images/3-https/3-test.png)

{{% notice info %}}
Đổi `ap-northeast-1` trong endpoint URL nếu bạn dùng Region khác.
{{% /notice %}}
