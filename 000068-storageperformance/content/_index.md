---
title : "Storage Performance Lab"
weight : 1
chapter : false
---


# Storage Performance Lab

#### Overview

Amazon S3 and Amazon EFS are built to scale out: they spread data over many servers, so a single client gets the best results when it sends **many requests at the same time**. In this lab you run the same transfer in different ways from one EC2 instance and measure the difference: one thread against many threads, one large object against many parts, one directory against many directories.

| Part | What you measure |
|---|---|
| Amazon S3 | Upload throughput with 1 to 20 threads, `aws s3 sync`, many small objects, and server-side copy |
| Amazon EFS | Creating many small files, I/O size and sync frequency, multi-threaded writes, and common copy tools (`rsync`, `cp`, `fpsync`, GNU `parallel`) |

#### Architecture

![Lab architecture](/images/architecture.png)

One CloudFormation template creates the whole lab: a VPC with one public subnet, an EC2 instance (Amazon Linux 2023, Graviton) with an EBS volume that already holds 2,000 sample files, an EFS file system mounted at `/efs`, and an S3 bucket reached through a gateway endpoint. You open a shell on the instance with **Session Manager**, so the instance has no inbound ports and no SSH key.

{{% notice info %}}
This is an updated version of the workshop. The new template replaces the Storage Immersion Day template: it no longer creates an IAM user or access keys, and the instance uses an IAM role that can only reach the lab bucket. All screenshots were taken in **Asia Pacific (Tokyo) ap-northeast-1**. You can use another Region, as long as you do every step in the same Region.
{{% /notice %}}

{{% notice warning %}}
Cost: the c7g.large instance, the EBS volumes, EFS and the test objects in S3 cost well under 1 USD if you finish the lab in about an hour. Do part 5 as soon as you finish.
{{% /notice %}}

The times in the screenshots come from one run. Your numbers will be different, but the trend should be the same.
