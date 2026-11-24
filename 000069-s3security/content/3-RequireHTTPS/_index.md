---
title : "Require HTTPS"
weight : 3
chapter : false
pre : " <b> 3. </b> "
---


#### Require HTTPS

Amazon S3 accepts requests over both HTTP and HTTPS. To make sure data is always encrypted in transit, add a bucket policy that denies any request where `aws:SecureTransport` is `false`.

1. In the S3 console, open your bucket and choose the **Permissions** tab.
2. Under **Bucket policy**, choose **Edit**.

![Bucket permissions](/images/3-https/3-permissions.png)

3. Paste the following policy and choose **Save changes**:

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

![Require HTTPS policy](/images/3-https/3-policy.png)

4. In the Session Manager terminal, read the object metadata over **HTTP** and then over **HTTPS**:

```bash
aws s3api head-object --key app1/file1 --endpoint-url http://s3.ap-northeast-1.amazonaws.com --bucket $bucket
aws s3api head-object --key app1/file1 --endpoint-url https://s3.ap-northeast-1.amazonaws.com --bucket $bucket
```

The HTTP request fails with **403 Forbidden**; the HTTPS request succeeds.

![HTTP denied, HTTPS allowed](/images/3-https/3-test.png)

{{% notice info %}}
Change `ap-northeast-1` in the endpoint URL if you use a different Region.
{{% /notice %}}
