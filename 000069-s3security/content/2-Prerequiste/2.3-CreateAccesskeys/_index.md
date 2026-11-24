---
title : "Review the IAM role of the instance"
weight : 3
chapter : false
pre : " <b> 2.3 </b> "
---


#### Review the IAM role of the instance

The original lab created an IAM user and stored its **access keys** on the instance. Long-term access keys are a common source of leaks, so this version uses an **IAM role** attached to the instance instead. The AWS CLI on the instance automatically receives temporary credentials from the instance profile.

1. Open the [IAM console](https://console.aws.amazon.com/iam/home) → **Roles** and search for `FCJ-S3Security-EC2-Role`.
2. On the **Permissions** tab you see two policies:
   + **AmazonSSMManagedInstanceCore** – lets Session Manager manage the instance.
   + **lab-bucket-access** (inline) – allows `s3:*` only on the lab bucket and its objects.

![IAM role permissions](/images/2-prerequisite/2.3-iam-role.png)

{{% notice info %}}
The role only has permissions on the lab bucket. Every **Deny** you add with a bucket policy in the next sections overrides this **Allow**, which is exactly what you will test.
{{% /notice %}}
