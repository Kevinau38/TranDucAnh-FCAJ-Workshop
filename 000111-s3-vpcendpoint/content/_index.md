---
title : "Secure Hybrid Access to S3 using VPC Endpoints"
weight : 1
chapter : false
---

# Secure Hybrid Access to S3 using VPC Endpoints

#### Overview

**AWS PrivateLink** gives your VPCs and your on-premises networks private connectivity to AWS services, without sending traffic over the public Internet.

In this lab you create, configure and test two kinds of VPC endpoint for **Amazon S3**:

+ **Gateway endpoint** – a route-table target for S3 (and DynamoDB). Instances inside the VPC reach S3 through a prefix-list route, with no Internet Gateway or NAT. There is no hourly charge.
+ **Interface endpoint** – elastic network interfaces (ENIs) with private IP addresses inside your subnets. Because they are reachable over private IPs, they can also be used from **on-premises** networks connected by VPN, Direct Connect or (in this lab) VPC peering. Interface endpoints are billed per hour and per GB.

![overview](/images/1-introduction/architecture.png)

#### Lab architecture

| Component | Purpose |
|---|---|
| **VPC Cloud** (10.10.0.0/16) | Two private subnets with no Internet access. Hosts the EC2 instance **Test-Gateway-Endpoint**, the S3 endpoints and the Route 53 Resolver inbound endpoint. |
| **VPC On-prem** (172.16.0.0/16) | Simulates a corporate data center. Hosts **Test-Interface-Endpoint** and the Route 53 Resolver outbound endpoint. |
| **VPC peering** | Simulates the private hybrid link (VPN / Direct Connect) between the two networks. |
| **SSM interface endpoints** | Let you open shells on the private instances with Session Manager – no SSH keys, no bastion. |

{{% notice info %}}
The original version of this workshop used a strongSwan VPN instance in us-east-1. This updated version replaces the VPN with **VPC peering** so the lab is faster and cheaper to run; the S3 endpoint concepts are exactly the same. All screenshots were taken in **Asia Pacific (Tokyo) ap-northeast-1** – you can use any Region, just keep every step in the same Region.
{{% /notice %}}
