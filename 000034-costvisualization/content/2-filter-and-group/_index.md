---
title : "Filter and group"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

#### Filter by service

Filters keep only the cost that you want to study. Here you look at the cost of Amazon EC2.

1. In **Report parameters** > **Filters**, open **Service**:
    - (1) Type `EC2` in the search box.
    - (2) Select **EC2 - Other** (EBS volumes and snapshots, NAT gateways, Elastic IP addresses, data transfer of EC2) and **EC2-Instances** (the instance hours). Leave **Includes** selected; **Excludes** removes the selected values instead.
    - (3) Choose **Apply**.

    ![Service filter](/images/2/2-1.png)

2. The report is updated:
    - (1) **Applied filters (1)**: the two services are listed under **Service**.
    - (2) The overview now shows only the cost of these 2 services.

    ![Filtered report](/images/2/2-2.png)

#### Group by usage type

3. Set **Group by** > **Dimension** to **Usage type** (1). The legend (2) now shows what you pay for inside EC2, for example `NatGateway-Hours`, `NatGateway-Bytes`, `EBS:VolumeUsage.gp2`, or `EBS:SnapshotUsage`. The prefix is the Region (`APS1` is Singapore, `USE2` is Ohio; no prefix means US East (N. Virginia)).

    ![Group by usage type](/images/2/2-3.png)

{{% notice tip %}}
In this account most of the "EC2" cost is **NAT gateway hours**, not instances. Grouping by usage type is the fastest way to find this kind of hidden cost.
{{% /notice %}}

#### Group by Region

4. Choose **Clear all** under **Filters**, set **Granularity** back to **Monthly**, then set **Dimension** to **Region** (1). The chart (2) compares the cost of each Region per month; **Global** is the cost of global services such as Amazon Route 53 or AWS WAF for Amazon CloudFront.

    ![Group by Region](/images/2/2-4.png)

Other useful dimensions: **Linked account** (in AWS Organizations), **Instance type**, **Purchase option** (On-Demand, Savings Plans, Spot), **Charge type** (usage, tax, credit, refund), and **Tag**.
