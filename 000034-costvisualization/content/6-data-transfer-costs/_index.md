---
title : "Data transfer costs"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---

#### Overview

Data transfer charges are easy to overlook when you design an architecture, and they appear in Cost Explorer under many services (for example **EC2 - Other**, Amazon S3, or Amazon CloudFront) with usage types that contain `DataTransfer`, `Bytes`, or `AWS-Out`. Filter on **Usage type group** > *Data Transfer* to see them together. The general rules:

| Traffic | Charge |
|---|---|
| From the internet into AWS | Free |
| From AWS to the internet | Charged per GB, by source Region and service. The first 100 GB per month, across all services and Regions, are free |
| Between Availability Zones in the same Region | Charged per GB in each direction |
| Inside one Availability Zone, using private IP addresses | Free |
| Between Regions | Charged per GB, by source Region |
| Through a NAT gateway | Hourly charge, plus a charge per GB processed |
| To Amazon S3 or DynamoDB through a gateway VPC endpoint | Free |
| Through an interface VPC endpoint (AWS PrivateLink) | Hourly charge per Availability Zone, plus a charge per GB processed |
| Through AWS Transit Gateway | Hourly charge per attachment, plus a charge per GB processed |
| Over a VPC peering connection in the same Availability Zone | Free; across Availability Zones, standard inter-AZ rates |
| AWS Site-to-Site VPN or AWS Direct Connect | Hourly charge for the connection, plus data transfer out of AWS |

#### Tips

- Access Amazon S3 and DynamoDB from private subnets through **gateway VPC endpoints** instead of a NAT gateway.
- Keep chatty components (for example an application server and its cache) in the same Availability Zone when high availability allows it.
- Use **Amazon CloudFront** to serve content to users: data transfer from the origin to CloudFront is free, and CloudFront has its own free tier.
- Avoid cross-Region traffic unless the business needs it, for example for disaster recovery.
- Estimate the data transfer of a new design with the [AWS Pricing Calculator](https://calculator.aws/).

Prices change and differ by Region: check the [Amazon EC2 data transfer pricing](https://aws.amazon.com/ec2/pricing/on-demand/#Data_Transfer) and the [Amazon VPC pricing](https://aws.amazon.com/vpc/pricing/).

#### Clean up

You deleted the only resource of this workshop, the saved report `fcj-monthly-cost-by-region`, in chapter 3. There is nothing else to clean up.
