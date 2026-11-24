---
title : "AWS Service Screener"
weight : 1
chapter : false
---

# AWS Service Screener

#### Overview

**Service Screener** is an open-source tool from AWS ([aws-samples/service-screener-v2](https://github.com/aws-samples/service-screener-v2)) that runs automated checks on your AWS account and compares the configuration of your resources with AWS and community best practices. It groups the findings by the pillars of the **AWS Well-Architected Framework**: Security, Reliability, Operational Excellence, Performance Efficiency and Cost Optimization.

The tool runs in **AWS CloudShell** and only calls read-only `describe`, `get` and `list` APIs, so it does not change your resources. The result is an HTML report that you open on your own computer.

![Service Screener dashboard](/images/4/3-dashboard.png)

In this workshop you:

1. Install Service Screener in CloudShell in the **Singapore (ap-southeast-1)** Region.
2. Create a small demo resource and scan it.
3. Download and read the report.
4. Send the results to the **AWS Well-Architected Tool**.
5. Clean up.

#### Architecture

The diagram shows what Service Screener touches in your account. It runs in CloudShell, reads the tagged demo bucket with read-only APIs, writes its results to a Well-Architected Tool workload, leaves one empty CloudFormation marker stack per run, and produces a report that you download to your computer.

![Workshop architecture](/images/architecture.png)

#### What you need

- An AWS account and a user or role with the permissions listed in [chapter 2](2-prepare-and-install/).
- About 45 minutes.

{{% notice info %}}
Running Service Screener is covered by the AWS Free Tier. If you are beyond the free tier, a run costs less than USD 0.01. The demo S3 bucket is empty and costs nothing.
{{% /notice %}}
