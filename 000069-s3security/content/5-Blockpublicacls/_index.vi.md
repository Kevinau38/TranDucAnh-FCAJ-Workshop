---
title : "Chặn ACL công khai"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---


#### Chặn ACL công khai

Bucket của bài lab dùng thiết lập object ownership **ObjectWriter**, nên ACL đang được bật và client có thể upload object kèm canned ACL như `public-read`. Trong phần này bạn sẽ thấy vì sao một câu lệnh **Allow** không đủ để ngăn điều đó, và câu lệnh **Deny** khắc phục ra sao.

#### Thử nghiệm 1 – câu lệnh Allow

1. Thay bucket policy bằng nội dung sau. Policy chỉ cho phép role của bài lab upload object khi ACL là `private`. Thay `<your-account-id>` và `<region>` bằng giá trị của bạn (có thể copy ARN của role trong IAM console).

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowPrivateAclOnly",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::<your-account-id>:role/FCJ-S3Security-EC2-Role-<region>"
            },
            "Action": [
                "s3:PutObject",
                "s3:PutObjectAcl"
            ],
            "Resource": "arn:aws:s3:::<your-bucket-name>/*",
            "Condition": {
                "StringEquals": {
                    "s3:x-amz-acl": "private"
                }
            }
        }
    ]
}
```

![Policy Allow](/images/5-acl/5-policy-allow.png)

2. Trong terminal, upload file với ACL mặc định rồi với `public-read`:

```bash
aws s3api put-object --key text01 --body textfile --bucket $bucket
aws s3api put-object --key text01 --body textfile --acl public-read --bucket $bucket
```

**Cả hai lệnh đều thành công.** IAM policy của role đã cho phép `s3:*` trên bucket, nên một câu lệnh **Allow** có điều kiện trong bucket policy không giới hạn được gì: request được chấp nhận nếu **bất kỳ** policy nào cho phép và không có policy nào từ chối.

![Upload public-read vẫn được phép](/images/5-acl/5-test-allow.png)

{{% notice warning %}}
Object `text01` lúc này có thể được đọc bởi bất kỳ ai biết URL của nó. Hãy upload lại với ACL mặc định (bước tiếp theo sẽ làm việc này) và không lưu dữ liệu thật trong bucket lab.
{{% /notice %}}

#### Thử nghiệm 2 – câu lệnh Deny

3. Thay bucket policy bằng câu lệnh **Deny** tường minh cho các canned ACL công khai:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyPublicAcls",
            "Effect": "Deny",
            "Principal": "*",
            "Action": [
                "s3:PutObject",
                "s3:PutObjectAcl"
            ],
            "Resource": "arn:aws:s3:::<your-bucket-name>/*",
            "Condition": {
                "StringEquals": {
                    "s3:x-amz-acl": [
                        "public-read",
                        "public-read-write",
                        "authenticated-read"
                    ]
                }
            }
        }
    ]
}
```

![Policy Deny ACL công khai](/images/5-acl/5-policy-deny.png)

4. Chạy lại các lệnh upload:

```bash
aws s3api put-object --key text01 --body textfile --acl public-read --bucket $bucket
aws s3api put-object --key text01 --body textfile --bucket $bucket
```

Lệnh upload `public-read` giờ bị từ chối, còn lệnh upload thông thường (ACL private) vẫn thành công và đưa object về trạng thái private.

![ACL công khai bị từ chối](/images/5-acl/5-test-deny.png)
