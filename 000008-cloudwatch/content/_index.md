---
title : "Amazon CloudWatch Workshop"
weight : 1
chapter : false
---

# Amazon CloudWatch Workshop

#### Overview

**Amazon CloudWatch** collects metrics and logs from your AWS resources and applications, lets you query and graph them, and raises alarms when something goes wrong. In this workshop you deploy three EC2 instances that run a sample workload and the **CloudWatch agent**, and then use CloudWatch to:

- explore **metrics**, combine them with **search expressions**, **math expressions** and **dynamic labels**;
- read **logs**, query them with **CloudWatch Logs Insights** and turn error lines into a metric with a **metric filter**;
- create an **alarm** that sends an email through **Amazon SNS** when errors spike;
- put everything on a **dashboard**.

#### Architecture

![Lab architecture](/images/architecture.png)

{{% notice info %}}
This is an updated version of the workshop. The CloudFormation template is new: it creates one public subnet without NAT gateways, uses **Amazon Linux 2023** (arm64, t4g.micro), gives the instances a least-privilege IAM role (**AmazonSSMManagedInstanceCore** and **CloudWatchAgentServerPolicy**) instead of AdministratorAccess, and installs the sample application from the template itself. All screenshots were taken in **Asia Pacific (Tokyo) ap-northeast-1**; you can use any Region, as long as you do every step in the same Region.
{{% /notice %}}

{{% notice warning %}}
Cost: three t4g.micro instances, a few CloudWatch custom metrics, one alarm and one dashboard cost well under 1 USD for a few hours. Do section 7 as soon as you finish.
{{% /notice %}}
