---
title : "Create the production VPC"
weight : 1
chapter : false
pre : " <b> 2.1 </b> "
---

1. Open the [Amazon VPC console](https://console.aws.amazon.com/vpc/), check that the Region is **Asia Pacific (Tokyo)**, and choose **Create VPC**.

2. In **VPC settings**:
   - (1) Choose **VPC and more**.
   - (2) Keep **Auto-generate** selected and enter `prod`. The console names the resources `prod-vpc`, `prod-subnet-private1-...` and so on.
   - (3) **IPv4 CIDR block**: `10.10.0.0/16`.

   ![VPC settings](/images/2-preparation/2.1-vpc-settings.png)

3. Scroll down and set:
   - (1) **Number of Availability Zones**: `1`.
   - (2) Open **Customize AZs** and choose `ap-northeast-1a` as the **First availability zone**. Write down the zone ID shown next to it (here `apne1-az4`). Both VPCs and the volume must use this zone.
   - (3) **Number of public subnets**: `0`.
   - (4) **Number of private subnets**: `1`.
   - (5) The preview shows one private subnet in `ap-northeast-1a`.

   ![Availability Zone and subnets](/images/2-preparation/2.1-vpc-az-subnets.png)

4. Scroll down to the bottom of the form:
   - **NAT gateways**: **None**.
   - **VPC endpoints**: **S3 Gateway**. The instances install packages from the Amazon Linux 2023 repositories, which are hosted in Amazon S3, through this endpoint.
   - Keep **Enable DNS hostnames** and **Enable DNS resolution** selected. The interface endpoints in step 2.3 need them.
   - Choose **Create VPC** and wait until the workflow shows **Success**.

{{% notice note %}}
Availability Zone names such as `ap-northeast-1a` can map to different physical zones in different AWS accounts. Inside one account the mapping is always the same, so using `ap-northeast-1a` for both VPCs puts both subnets in the same zone.
{{% /notice %}}
