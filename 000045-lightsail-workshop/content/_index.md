---
title : "Amazon Lightsail Workshop"
weight : 1
chapter : false
---

# Amazon Lightsail Workshop

#### Overview

**Amazon Lightsail** is the simplest way to run a website or a small application on AWS. A Lightsail plan bundles a virtual server, SSD storage, data transfer, a public IPv4 address and DNS management for a fixed monthly price, so you do not need to design a VPC, security groups or EBS volumes before you start.

In this workshop you build two applications on Lightsail in the **Singapore (ap-southeast-1)** Region and then operate them:

| Application | How it is deployed |
|---|---|
| **WordPress** blog | An Ubuntu 24.04 **OS blueprint**, a launch script, and a Lightsail **managed MySQL database** |
| **PrestaShop** online store | A ready-made **application blueprint** |

After both applications are running, you secure them with firewall rules, back them up with snapshots, move WordPress to a larger plan, and create a CPU alarm.

![Lightsail instances](/images/3/4-instances.png)

#### Architecture

The diagram shows what you build. Both applications run as Lightsail instances behind static IP addresses. WordPress stores its data in a Lightsail managed MySQL database, so in chapter 6 you can move the blog to a larger instance created from a snapshot without losing content.

![Workshop architecture](/images/architecture.png)

#### What you need

- An AWS account and a user or role that can use **Amazon Lightsail** and **AWS CloudShell**.
- About 90 minutes.

{{% notice warning %}}
Lightsail resources are billed by the hour from the moment they are created, even when an instance is stopped. This lab uses a USD 15/month database and instances of USD 7 and USD 12/month. If you finish in 2–3 hours, the cost is well under USD 1. Do the [Clean up](8-clean-up/) chapter as soon as you finish.
{{% /notice %}}
