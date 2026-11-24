---
title : "Savings Plans recommendations"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

#### Overview

AWS looks at your On-Demand usage in the past 7, 30, or 60 days and recommends the hourly commitment that would save you the most. The recommendations are free and are updated at least once a day.

1. In the **Savings Plans** menu, choose **Recommendations**:
    - (1) **Savings Plans type**: Compute, EC2 Instance, SageMaker, or Database.
    - (2) **Savings Plans term**: 1 or 3 years.
    - (3) **Payment option**: All, Partial, or No upfront.
    - (4) **Based on the past**: the period of usage used for the calculation.

    ![Recommendation parameters](/images/2/2-1.png)

2. Scroll down to the results:
    - (1) **Current monthly on-demand spend**, **Estimated monthly spend**, and **Estimated monthly savings** compare your bill before and after the recommended purchase.
    - (2) In the **Recommended Savings Plans** table, you can select a recommendation and choose **Add Savings Plans to cart**.
    - (3) In a lab account, the table is usually empty: AWS shows no recommendation when there is no savings opportunity, or when your average On-Demand spend is below **USD 0.10 per hour**.

    ![Recommendations](/images/2/2-2.png)

{{% notice tip %}}
Before you buy, use **Purchase Analyzer** in the same menu to model a commitment: it estimates the cost, coverage, and utilization of a plan against your real usage.
{{% /notice %}}
