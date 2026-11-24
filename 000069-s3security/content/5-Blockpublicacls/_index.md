---
title : "Block public ACLs"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---


#### Block public ACLs

The lab bucket uses the **ObjectWriter** object ownership setting, so ACLs are enabled and a client can upload an object with a canned ACL such as `public-read`. In this section you see why an **Allow** statement is not enough to stop that, and how a **Deny** statement fixes it.

#### Attempt 1 – an Allow statement

1. Replace the bucket policy with the following one. It allows the lab role to upload objects only when the ACL is `private`. Replace `<your-account-id>` and `<region>` with your values (you can copy the role ARN from the IAM console).

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

![Allow policy](/images/5-acl/5-policy-allow.png)

2. In the terminal, upload the file with the default ACL and then with `public-read`:

```bash
aws s3api put-object --key text01 --body textfile --bucket $bucket
aws s3api put-object --key text01 --body textfile --acl public-read --bucket $bucket
```

**Both uploads succeed.** The role's IAM policy already allows `s3:*` on the bucket, so a bucket-policy **Allow** with a condition does not restrict anything: access is granted if **any** policy allows it and none denies it.

![Public-read upload still allowed](/images/5-acl/5-test-allow.png)

{{% notice warning %}}
The object `text01` is now readable by anyone who knows its URL. Re-upload it with the default ACL (the next step does this) and do not store real data in the lab bucket.
{{% /notice %}}

#### Attempt 2 – a Deny statement

3. Replace the bucket policy with an explicit **Deny** for public canned ACLs:

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

![Deny public ACLs policy](/images/5-acl/5-policy-deny.png)

4. Run the uploads again:

```bash
aws s3api put-object --key text01 --body textfile --acl public-read --bucket $bucket
aws s3api put-object --key text01 --body textfile --bucket $bucket
```

The `public-read` upload is now denied, while the normal upload (private ACL) still succeeds and resets the object to private.

![Public ACL denied](/images/5-acl/5-test-deny.png)
