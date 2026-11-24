---
title : "Require SSE-S3 encryption"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---


#### Require SSE-S3 encryption

S3 now encrypts every new object with SSE-S3 by default. Some organizations still require clients to **explicitly** request encryption, so that an upload without the header is rejected. You can enforce this with a bucket policy on the `s3:x-amz-server-side-encryption` condition key.

1. Open **Permissions** → **Bucket policy** → **Edit** again.
2. Replace the policy with the following one and choose **Save changes**:

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

![Require SSE-S3 policy](/images/4-sse/4-policy.png)

3. In the terminal, create a test file and upload it without and then with the encryption header:

```bash
echo "123456789abcdefg" > textfile
aws s3api put-object --key text01 --body textfile --bucket $bucket
aws s3api put-object --key text01 --body textfile --server-side-encryption AES256 --bucket $bucket
```

The first upload fails with **AccessDenied ... with an explicit deny in a resource-based policy**; the second one succeeds and returns `"ServerSideEncryption": "AES256"`.

![Upload without SSE denied](/images/4-sse/4-test.png)
