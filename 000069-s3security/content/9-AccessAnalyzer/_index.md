---
title : "Use IAM Access Analyzer for S3"
weight : 9
chapter : false
pre : " <b> 9. </b> "
---


#### Use IAM Access Analyzer for S3

**IAM Access Analyzer** uses automated reasoning to find resources – S3 buckets, IAM roles, KMS keys and more – that can be accessed from **outside your zone of trust** (your account or organization). For S3 it analyzes bucket policies, ACLs, access points and Block Public Access settings, and creates a **finding** for every bucket shared publicly or with another account.

#### Create an analyzer

1. Open the [IAM console](https://console.aws.amazon.com/iam/home) → **Access analyzer** → **Analyzer settings** → **Create analyzer**.
2. Keep **Resource analysis – External access** (no additional cost), enter the name `FCJ-S3-Analyzer`. The zone of trust is the **current account**.

![Create analyzer](/images/9-analyzer/9-create.png)

3. Choose **Create analyzer**.

![Create analyzer button](/images/9-analyzer/9-create-btn.png)

#### Review the findings

4. The **External access** page opens. The first scan takes a few minutes. Filter the findings with **Resource Type = AWS::S3::Bucket**.

![S3 findings](/images/9-analyzer/9-findings.png)

Because you removed the public ACL and the bucket policies in the previous sections, the lab bucket has **no active finding**: it is not shared with the public or with any other AWS account.

{{% notice info %}}
How to read a finding:
+ **Active** – the bucket is accessible from outside the zone of trust. If the access is intended, **archive** the finding; if not, remove the policy statement, ACL grant or access point that causes it.
+ **Resolved** – the access was removed and the analyzer no longer sees it.
+ **Archived** – you marked the access as intended.

The S3 console also shows these results on the **Permissions** tab of a bucket (**Access finding**) and in the **External access summary** of the bucket list.
{{% /notice %}}

{{% notice tip %}}
In a shared account you may also see findings for IAM roles that belong to other people. Do not change them; filter on your own resource type or resource name.
{{% /notice %}}
