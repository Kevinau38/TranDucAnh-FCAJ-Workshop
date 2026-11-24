---
title : "Configure S3 Block Public Access"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---


#### Configure S3 Block Public Access

Writing deny policies for every case is error-prone. **S3 Block Public Access** gives you four simple switches that override any policy or ACL that would make data public. In this section you replace the bucket policy from section 5 with the **new ACLs** setting.

1. On the **Permissions** tab, under **Bucket policy**, choose **Delete**, type `confirm` and choose **Delete**.

![Delete bucket policy](/images/6-bpa/6-delete-policy.png)

2. Under **Block public access (bucket settings)**, choose **Edit**.
3. Select **Block public access to buckets and objects granted through *new* access control lists (ACLs)**, keep the two policy-related settings selected, then choose **Save changes**. Type `confirm` when asked.

![Block new public ACLs](/images/6-bpa/6-bpa-edit.png)

4. In the terminal, try the public upload again:

```bash
aws s3api put-object --key text01 --body textfile --acl public-read --bucket $bucket
```

The request is denied with **public ACLs are prevented by the BlockPublicAcls setting in S3 Block Public Access** – no bucket policy needed.

![Blocked by S3 Block Public Access](/images/6-bpa/6-test.png)

{{% notice info %}}
In production, keep **Block all public access** turned on at both the account and bucket level unless you have a specific reason not to. To prepare for section 8, edit the setting again and **clear** the *new ACLs* option you just selected, so that you can deliberately create a public ACL and detect it.
{{% /notice %}}
