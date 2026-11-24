---
title : "Savings Plans, Reserved Instances and Reserved DB Instances"
weight : 1
chapter : false
---

# Savings Plans, Reserved Instances and Reserved DB Instances

#### Overview

On-Demand pricing is simple: you pay by the second or the hour and stop paying when you stop the resource. For workloads that run all the time, AWS offers lower prices in exchange for a **commitment of 1 or 3 years**. This workshop explains the three commitment models for compute and databases, and walks you through the purchase pages in the AWS console:

| Model | What you commit to | Applies to |
|---|---|---|
| **Savings Plans** | An amount of money per hour (for example USD 0.10/hour) | EC2, Fargate, Lambda, SageMaker AI, and managed databases, depending on the plan type |
| **EC2 Reserved Instances** | A specific instance type, platform, and Region | Amazon EC2 |
| **Reserved DB instances** | A specific DB engine, instance class, and Region | Amazon RDS and Amazon Aurora |

{{% notice warning %}}
This workshop never completes a purchase. Savings Plans, Reserved Instances, and reserved DB instances are commitments for 1 or 3 years: once the order is submitted, you pay for every hour of the term whether you use it or not. You stop at the cart, then empty it.
{{% /notice %}}

#### What you need

- An AWS account and a user or role that can open **Billing and Cost Management**, the Amazon EC2 console, and the Amazon RDS console.
- No resources are created, so the workshop has **no cost**.
