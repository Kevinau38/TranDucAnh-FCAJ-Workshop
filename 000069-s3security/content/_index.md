---
title : "S3 Security Best Practices"
weight : 1
chapter : false
---

# S3 Security Best Practices

#### Overview

In this lab you apply a set of security best practices to protect data stored in **Amazon S3**. You work from an EC2 instance that has an IAM role, and you use **bucket policies**, **ACLs**, **S3 Block Public Access**, a **VPC gateway endpoint**, an **AWS Config** rule and **IAM Access Analyzer** to control and audit who can reach your bucket.

![Lab architecture](/images/1-introduction/architecture.png)

{{% notice info %}}
This is an updated version of the workshop. The lab environment is now created by one CloudFormation template, and the EC2 instance uses an **IAM role** and **Session Manager** instead of an IAM user, access keys and SSH. All screenshots were taken in **Asia Pacific (Tokyo) ap-northeast-1**; you can use any Region, as long as you keep every step in the same Region.
{{% /notice %}}
