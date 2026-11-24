---
title : "Savings Plans and Reserved Instances"
weight : 2
chapter : false
pre : " <b> 1.2. </b> "
---

#### Comparing Savings Plans and Reserved Instances

Both models lower the price of steady usage in exchange for a 1- or 3-year commitment. Savings Plans commit to **money per hour**; Reserved Instances commit to **a specific instance configuration**.

| | Compute Savings Plans | EC2 Instance Savings Plans | Convertible RIs | Standard RIs |
|---|---|---|---|---|
| Discount compared to On-Demand (up to) | 66% | 72% | 66% | 72% |
| Applies to any instance family | ✓ | — | Exchange needed | — |
| Applies to any instance size | ✓ | ✓ | ✓ (Regional scope) | ✓ (Regional scope) |
| Applies to any operating system or tenancy | ✓ | ✓ | Exchange needed | — |
| Applies across Regions | ✓ | — | — | — |
| Applies to AWS Fargate and AWS Lambda | ✓ | — | — | — |
| Can reserve capacity in an Availability Zone | — | — | ✓ (zonal) | ✓ (zonal) |
| Can be sold on the Reserved Instance Marketplace | — | — | — | ✓ |
| Term | 1 or 3 years | 1 or 3 years | 1 or 3 years | 1 or 3 years |

{{% notice info %}}
AWS recommends Savings Plans for most new commitments on compute: you get the same discount as Reserved Instances without being tied to one instance configuration. Reserved Instances are still useful when you need **capacity reservation** in an Availability Zone, or for services that Savings Plans do not cover.
{{% /notice %}}
