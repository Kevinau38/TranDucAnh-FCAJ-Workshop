---
title : "Create the VPC"
weight : 2
chapter : false
pre : " <b> 2.2 </b> "
---

You create a VPC in two Availability Zones. Each zone has one public subnet, which is used only by the Application Load Balancer, and one private subnet, which is used by the EC2 instances and the RDS database. There is **no NAT gateway**: the private subnets have no route to the internet.

1. Open the [Amazon VPC console](https://console.aws.amazon.com/vpc/) and choose **Create VPC**.

2. In **VPC settings**:
   - (1) Choose **VPC and more**.
   - (2) Keep **Auto-generate** selected and enter `wordpress`. The console names all resources with this prefix, for example `wordpress-vpc`.
   - Keep the IPv4 CIDR block `10.0.0.0/16`.

   ![VPC settings](/images/2-prerequisite/2.2-vpc-settings.png)

3. Scroll down and set the following:
   - Keep **Number of Availability Zones** = `2` and **Number of public subnets** = `2`.
   - (1) **Number of private subnets**: `2`.
   - (2) **NAT gateways**: **None**.
   - (3) **VPC endpoints**: **S3 Gateway**. The private subnets reach Amazon S3 through this endpoint.
   - Keep **Enable DNS hostnames** and **Enable DNS resolution** selected. The interface endpoints in step 2.4 need them.
   - (4) Choose **Create VPC**.

   ![NAT gateway and S3 endpoint](/images/2-prerequisite/2.2-vpc-nat-endpoint.png)

4. Wait until the workflow shows (1) **Success**. The details show that (2) the S3 endpoint was created and (3) associated with the private route tables.

   ![VPC created](/images/2-prerequisite/2.2-vpc-success.png)

{{% notice tip %}}
A gateway endpoint for Amazon S3 has no hourly charge. The Amazon Linux 2023 package repositories are also hosted in Amazon S3, so the private instances can install packages with `dnf` through this endpoint.
{{% /notice %}}
