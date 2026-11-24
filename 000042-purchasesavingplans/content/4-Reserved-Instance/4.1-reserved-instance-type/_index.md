---
title : "Reserved Instance types"
weight : 1
chapter : false
pre : " <b> 4.1. </b> "
---

#### Offering classes

| | Standard RI | Convertible RI |
|---|---|---|
| Discount compared to On-Demand (up to) | 72% | 66% |
| Change Availability Zone, scope, network platform, or instance size (same family, Linux/UNIX) | ✓ (modify) | ✓ (modify) |
| Change instance family, operating system, tenancy, or payment option | — | ✓ (exchange for another Convertible RI of equal or greater value) |
| Sell on the Reserved Instance Marketplace | ✓ | — |

#### Scope

- **Regional**: the discount applies to matching usage in any Availability Zone of the Region and, for Linux/UNIX with default tenancy, to any size in the same instance family (size flexibility). There is no capacity reservation.
- **Zonal**: the discount applies only in one Availability Zone, and AWS **reserves capacity** for the instance there.

#### Payment options and terms

RIs use the same payment options as Savings Plans (**All Upfront**, **Partial Upfront**, **No Upfront**) and a term of **1 year** or **3 years**. You can also buy Standard RIs with a shorter remaining term from other customers on the **Reserved Instance Marketplace**.

{{% notice warning %}}
With **No Upfront** and **Partial Upfront**, you are billed for every hour of the term, whether or not a matching instance is running.
{{% /notice %}}
