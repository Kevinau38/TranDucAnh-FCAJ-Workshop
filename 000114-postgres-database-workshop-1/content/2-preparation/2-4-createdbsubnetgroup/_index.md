---
title : "Create DB Subnet Group"
weight : 4
chapter : false
pre : " <b> 2.4. </b> "
---

#### Overview

A **DB subnet group** tells Amazon RDS which subnets it can place your database in. You choose the subnet group when you create the RDS instance, not a single subnet.

In this step, you create `rdspg-lab-subnet-group` with the **two private subnets** from step 2.1:

| Subnet | Availability Zone | CIDR |
|---|---|---|
| `rdspg-lab-subnet-private1-ap-southeast-1a` | `ap-southeast-1a` | `10.0.128.0/20` |
| `rdspg-lab-subnet-private2-ap-southeast-1b` | `ap-southeast-1b` | `10.0.144.0/20` |

{{% notice info %}}
A DB subnet group must contain subnets in **at least two Availability Zones**, even if you do not enable Multi-AZ. This lets RDS place a standby or move the database to another AZ when needed.
{{% /notice %}}

#### Create the DB subnet group

1. Open the [Amazon RDS console](https://console.aws.amazon.com/rds/):
    - (1) Choose **Subnet groups**.
    - (2) Choose **Create DB subnet group**.

    ![Create DB subnet group](/images/2-preparation/2.4/1.png)

2. In **Subnet group details**:
    - (1) **Name**: `rdspg-lab-subnet-group`
    - (2) **Description**: `Private subnets for RDS PostgreSQL`
    - (3) **VPC**: `rdspg-lab-vpc`

    ![Subnet group details](/images/2-preparation/2.4/2.png)

3. In **Add subnets**:
    - (1) **Availability Zones**: keep only `ap-southeast-1a` and `ap-southeast-1b`. Remove any other AZ that is selected by default.
    - (2) **Subnets**: choose the two private subnets, `10.0.128.0/20` and `10.0.144.0/20`.
    - (3) Choose **Create**.

    ![Add subnets](/images/2-preparation/2.4/3.png)

{{% notice warning %}}
Do not choose the public subnets (`10.0.0.0/20` and `10.0.16.0/20`). The database must stay in private subnets.
{{% /notice %}}

{{% notice note %}}
The console shows a message that **Multi-AZ DB clusters** need three subnets in three AZs. This workshop uses a single DB instance, so two subnets in two AZs are enough.
{{% /notice %}}

4. The DB subnet group `rdspg-lab-subnet-group` is created in `rdspg-lab-vpc`.

    ![DB subnet group created](/images/2-preparation/2.4/4.png)
