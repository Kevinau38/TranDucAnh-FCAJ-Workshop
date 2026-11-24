---
title : "Introduction"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---


#### S3 security practice

By default, an S3 bucket and its objects are private: only the account that owns them can access them. You can then grant access with a combination of:

+ **IAM policies** attached to users and roles – what an identity is allowed to do.
+ **Bucket policies** – resource-based policies that apply to every request to the bucket, from any principal.
+ **Access control lists (ACLs)** – a legacy mechanism that grants basic read/write permissions on a bucket or object. AWS recommends keeping ACLs disabled unless you really need them.
+ **S3 Block Public Access** – account-level and bucket-level guardrails that override policies and ACLs that would make data public.

In this lab you will:

| Section | What you do |
|---|---|
| 3 | Deny every request that does not use HTTPS |
| 4 | Deny uploads that do not use SSE-S3 encryption |
| 5 | Stop clients from uploading objects with public ACLs |
| 6 | Use S3 Block Public Access instead of a bucket policy |
| 7 | Allow object access only through an S3 VPC gateway endpoint |
| 8 | Detect a public bucket with an AWS Config managed rule |
| 9 | Review external access to S3 with IAM Access Analyzer |

![Lab architecture](/images/1-introduction/architecture.png)

#### Cost

+ The EC2 instance is a **t3.micro** (free-tier eligible in many accounts) and the S3 gateway endpoint has no hourly charge.
+ AWS Config charges per recorded configuration item and per rule evaluation; recording only S3 buckets for a short lab costs a few cents.
+ An IAM Access Analyzer **external access** analyzer has no additional cost.
+ Complete the [clean up](../10-cleanup/) section as soon as you finish.
