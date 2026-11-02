---
title : "Create a VPC"
weight : 1
chapter : false
pre : " <b> 2.1. </b> "
---

#### Overview

In this step, you create the network for the whole workshop: a VPC with **2 public subnets** and **2 private subnets**, spread across **2 Availability Zones (AZs)**.

| Resource | Subnet | Reason |
|---|---|---|
| Bastion host | Public | You connect to it from your computer over SSH |
| App server | Private | It does not need to be reachable from the internet |
| RDS PostgreSQL | Private (2 AZs) | A DB subnet group requires subnets in at least 2 AZs |

{{% notice info %}}
This workshop does **not** use a NAT gateway, because a NAT gateway is charged per hour. The App server in the private subnet still installs packages: Amazon Linux 2023 repositories are hosted on Amazon S3, and the **S3 Gateway endpoint** gives the private subnets a route to S3 at no additional cost.
{{% /notice %}}

#### Create the VPC

1. Open the [Amazon VPC console](https://console.aws.amazon.com/vpc/):
    - (1) Choose **Your VPCs**.
    - (2) Choose **Create VPC**.

    ![Create VPC](/images/2-preparation/2.1/1.png)

2. In **VPC settings**:
    - (1) For **Resources to create**, choose **VPC and more**.
    - (2) For **Name tag auto-generation**, keep **Auto-generate** selected and enter `rdspg-lab`.
    - (3) For **IPv4 CIDR block**, enter `10.0.0.0/16`.
    - (4) For **IPv6 CIDR block**, choose **No IPv6 CIDR block**.
    - (5) Keep **Tenancy** as **Default**. Leave **Encryption settings** unchanged.

    ![VPC settings](/images/2-preparation/2.1/2.png)

3. Configure the subnets and NAT gateways:
    - (1) **Number of Availability Zones (AZs)**: `2`
    - (2) **Number of public subnets**: `2`
    - (3) **Number of private subnets**: `2`
    - (4) **NAT gateways**: **None**

    ![Subnets and NAT gateways](/images/2-preparation/2.1/3.png)

4. Finish the configuration:
    - (1) **VPC endpoints**: **S3 Gateway**
    - (2) **DNS options**: keep both **Enable DNS hostnames** and **Enable DNS resolution** selected.
    - (3) Choose **Create VPC**.

    ![VPC endpoints and DNS options](/images/2-preparation/2.1/4.png)

{{% notice note %}}
Both DNS options are required. You connect to Amazon RDS through its **endpoint**, which is a DNS name such as `xxx.rds.amazonaws.com`, not an IP address.
{{% /notice %}}

5. Wait until every step shows **Success**:
    - (1) Choose **View VPC**.

    ![Create VPC workflow](/images/2-preparation/2.1/5.png)

#### Verify the network with the Resource map

6. On the VPC details page, open the **Resource map** tab and turn on **Show all details**.

    ![Resource map](/images/2-preparation/2.1/6.png)

| Route table | Associated subnets | Routes | Result |
|---|---|---|---|
| `rdspg-lab-rtb-public` | 2 public subnets | `local`, `0.0.0.0/0` → Internet gateway | Public: inbound and outbound internet access |
| `rdspg-lab-rtb-private1-...` | Private subnet in AZ 1a | `local`, Amazon S3 → S3 Gateway endpoint | Private: no internet access, can reach S3 |
| `rdspg-lab-rtb-private2-...` | Private subnet in AZ 1b | `local`, Amazon S3 → S3 Gateway endpoint | Private: no internet access, can reach S3 |
| Main route table | None | `local` | Created automatically with every VPC, not used here |

{{% notice tip %}}
A subnet is **public** only because its route table has a route to an Internet gateway. There is no "public" checkbox on the subnet itself.
{{% /notice %}}
