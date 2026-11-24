---
title : "Use an AWS Config rule to detect a public bucket"
weight : 8
chapter : false
pre : " <b> 8. </b> "
---


#### Use an AWS Config rule to detect a public bucket

**AWS Config** records the configuration of your resources and evaluates it against rules. The managed rule **s3-bucket-public-read-prohibited** checks the Block Public Access settings, the bucket policy and the bucket ACL, and flags any bucket that allows public read access.

#### Set up AWS Config

{{% notice info %}}
If AWS Config is already recording in this Region, skip to **Add the rule**. Do not change a recorder that other people use in a shared account.
{{% /notice %}}

1. Open the [AWS Config console](https://console.aws.amazon.com/config/home) and choose **Get started**.

![AWS Config home](/images/8-config/8-config-home.png)

2. In **Recording strategy**, choose **Specific resource types**, to keep the cost low.

![Specific resource types](/images/8-config/8-settings.png)

3. In **Resource type**, choose **AWS S3 Bucket** with frequency **Continuous**. Keep **Use an existing AWS Config service-linked role**.

![Record S3 buckets](/images/8-config/8-resource-type.png)

4. In **Delivery channel**, choose **Create a bucket** and enter a name such as `fcj-config-lab-<suffix>`, then choose **Next**. Skip the rules page with **Next** and choose **Confirm**.

![Delivery channel bucket](/images/8-config/8-delivery.png)

#### Add the rule

5. In the AWS Config console, choose **Rules** → **Add rule**. Keep **Add AWS managed rule**, search for `s3-bucket-public-read-prohibited`, select it and choose **Next**.

![Select the managed rule](/images/8-config/8-rule-select.png)

6. Keep the default name and settings, choose **Next**, then **Save**.

![Configure the rule](/images/8-config/8-rule-config.png)

#### Make the bucket public on purpose

7. In the S3 console, open the lab bucket → **Permissions** → **Access control list (ACL)** → **Edit**.
8. For **Everyone (public access)**, select **Objects – List**, select **I understand the effects of these changes on my objects and buckets** and choose **Save changes**.

![Grant public list access](/images/8-config/8-acl-edit.png)

#### Evaluate the rule

9. Back in AWS Config, open the rule **s3-bucket-public-read-prohibited**, choose **Actions** → **Re-evaluate**.

![Re-evaluate the rule](/images/8-config/8-reevaluate.png)

10. After about a minute, scroll to **Resources in scope** and choose **All**. The lab bucket is **Noncompliant** with the annotation *The S3 bucket ACL allows public read access*, while the private Config delivery bucket is **Compliant**.

![Noncompliant bucket](/images/8-config/8-noncompliant.png)

11. Remove the public grant: edit the bucket ACL again, clear **Everyone (public access) – List** and choose **Save changes**.

{{% notice tip %}}
In accounts protected by guardrails (for example AWS Organizations with automatic remediation), the public ACL may be reverted and Block Public Access turned back on within a few minutes. That is the behavior you want in production; the rule result you captured before the remediation is enough for this lab.
{{% /notice %}}
