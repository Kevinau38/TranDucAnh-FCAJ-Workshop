---
title : "Savings Plans types"
weight : 1
chapter : false
pre : " <b> 1.1. </b> "
---

#### Four types of Savings Plans

1. Open the [Savings Plans console](https://console.aws.amazon.com/costmanagement/home#/savings-plans/overview) (**Billing and Cost Management** > **Savings Plans** > **Overview**):
    - (1) The **Savings Plans** menu: Overview, Inventory, Recommendations, Purchase Analyzer, Utilization and Coverage reports, Purchase Savings Plans, and Cart.
    - (2) AWS offers four types of Savings Plans.
    - (3) **Purchase a Savings Plan** opens the purchase page that you use in chapter 3.

    ![Savings Plans overview](/images/1/1-1.png)

| Type | Discount (up to) | Covers | Flexibility |
|---|---|---|---|
| **Compute Savings Plans** | 66% | EC2 instances, AWS Fargate, AWS Lambda | Any instance family, size, Availability Zone, Region, operating system, or tenancy |
| **EC2 Instance Savings Plans** | 72% | EC2 instances | One instance family in one Region (for example M7g in Singapore); any size, operating system, or tenancy in that family |
| **SageMaker AI Savings Plans** | 64% | Amazon SageMaker AI instances | Any instance family, size, Region, or component (notebook, training, inference) |
| **Database Savings Plans** | 35% | Amazon Aurora, Amazon RDS, Amazon DynamoDB, Amazon ElastiCache, Amazon DocumentDB, Amazon Neptune, Amazon OpenSearch Service, and other managed databases | Any supported engine, instance family, size, deployment option, or Region |

{{% notice tip %}}
Compute Savings Plans give the most flexibility; EC2 Instance Savings Plans give the largest discount if you know which instance family you will run. Database Savings Plans are available for a 1-year term with No Upfront payment only.
{{% /notice %}}
