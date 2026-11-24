---
title : "VPC endpoint policies"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---


A **VPC endpoint policy** is an IAM resource policy attached to the endpoint. It controls which principals, actions and resources can be reached **through that endpoint**. It does not replace IAM or bucket policies – a request must be allowed by all of them. The default policy allows full access.

In this section you restrict **s3-interface-endpoint** so that clients coming from on-premises can only reach one approved bucket. Gateway endpoint policies work exactly the same way.

#### Create a second bucket

1. In the S3 console create a new bucket with the same name as your first bucket plus `-2`.

![bucket 2](/images/5-policy/5-bucket2-name.png)
![bucket 2 created](/images/5-policy/5-bucket2-done.png)

#### Edit the endpoint policy

1. Open **VPC console → Endpoints → s3-interface-endpoint → Policy**. The current policy allows everything. Choose **Edit Policy**.

![default policy](/images/5-policy/5-policy-default.png)

2. Choose **Custom** and paste the policy below (replace the bucket name), then choose **Save**.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowOnlyBucket2",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::<your-bucket-name>-2",
        "arn:aws:s3:::<your-bucket-name>-2/*"
      ]
    }
  ]
}
```

![edit policy](/images/5-policy/5-policy-edit.png)

#### Test the policy

In the **Test-Interface-Endpoint** session run:

```bash
aws s3 cp testfile2.xyz s3://<your-bucket-name>-2 --region <your-region>
aws s3 ls s3://<your-bucket-name>-2 --region <your-region>
aws s3 ls s3://<your-bucket-name> --region <your-region>
```

{{% notice note %}}
Endpoint policy changes can take a minute or two to take effect. If the last command still succeeds, wait a moment and run it again.
{{% /notice %}}

+ Access to **&lt;your-bucket-name&gt;-2** succeeds.
+ Access to the first bucket fails with **AccessDenied ... because no VPC endpoint policy allows the s3:ListBucket action**, even though the instance role has `AmazonS3FullAccess`.

![test policy](/images/5-policy/5-test.png)
