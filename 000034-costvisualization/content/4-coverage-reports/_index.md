---
title : "Coverage reports"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

#### Coverage and utilization

When you use Savings Plans or Reserved Instances (see workshop [000042](https://000042.awsfcaj.com/)), two questions matter:

| Report | Question | Target |
|---|---|---|
| **Coverage** | How much of my eligible usage is covered by a commitment? The rest is billed On-Demand. | High, for steady workloads |
| **Utilization** | How much of my commitment do I actually use? The unused part is wasted. | Close to 100% |

#### Savings Plans coverage

1. In **Cost Explorer Saved Reports**, open the default **Coverage report** (type *Savings Plans coverage*):
    - (1) The breadcrumb shows that the report belongs to **Savings Plans**.
    - (2) **On-Demand spend not covered**, **Average coverage**, and **Potential monthly savings**. Here the account has no Savings Plan, so the coverage is empty.
    - (3) **Coverage target (%)** draws a dashed target line on the chart; **Granularity** is daily or monthly.

    ![Savings Plans coverage](/images/4/4-1.png)

2. Scroll down:
    - (1) **Savings Plans coverage breakdown**: for each service, instance family, and Region, the spend covered by Savings Plans and the On-Demand spend. Lambda and DynamoDB appear because Compute and Database Savings Plans can cover them.
    - (2) Filter the report by linked account, Region, instance family, service, or cost category.

    ![Coverage breakdown](/images/4/4-2.png)

#### Reserved Instances coverage

3. Go back to the report library and open **RI Coverage**:
    - (1) Reservation reports are per **Service**: EC2, RDS, ElastiCache, OpenSearch Service, Redshift, and others.
    - (2) This account has no reservation, so there is no coverage data.

    ![RI coverage](/images/4/4-3.png)

The **Utilization report** (Savings Plans) and **RI Utilization** work the same way. Use them together with the recommendations of chapter 2 in workshop 000042 before buying a new commitment.
