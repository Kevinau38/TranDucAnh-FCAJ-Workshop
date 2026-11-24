---
title : "Purchase a Savings Plan"
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

#### Overview

In this chapter you configure a Compute Savings Plan on the purchase page, add it to the cart, and then remove it. You do not submit the order.

{{% notice warning %}}
This workshop never completes a purchase. Savings Plans, Reserved Instances, and reserved DB instances are commitments for 1 or 3 years: once the order is submitted, you pay for every hour of the term whether you use it or not. You stop at the cart, then empty it.
{{% /notice %}}

#### Configure the plan

1. In the **Savings Plans** menu, choose **Purchase Savings Plans**:
    - (1) **Savings Plans type**: choose **Compute Savings Plans**.
    - (2) **Term**: choose **1-year**.

    ![Purchase details](/images/3/3-1.png)

2. Fill in the commitment:
    - (1) **Hourly commitment**: enter **0.001** (USD per hour). The amount is at Savings Plans rates, not On-Demand rates.
    - (2) **Payment option**: choose **No upfront**.
    - (3) **Start date - optional**: leave it empty to start the plan as soon as the order is submitted, or set a future date and time (UTC) to **queue** the plan.
    - (4) **Purchase summary** shows the upfront cost, the monthly payment, and the total cost of the commitment: USD 0.001 × 8,760 hours ≈ USD 8.76 for one year.
    - (5) Choose **Add to cart**.

    ![Purchase commitment](/images/3/3-2.png)

3. Because the account has no steady usage, AWS warns that you may be **overcommitting**:
    - (1) The warning compares your commitment with your usage in the last 30 days.
    - (2) Choose **Confirm add to cart**. Adding to the cart does not buy anything.

    ![Overcommitting warning](/images/3/3-3.png)

#### Review and empty the cart

4. The **Cart** page:
    - (1) The plan in the cart: type, term, payment option, start date, commitment, and payments.
    - (2) **Summary**: total commitment, commitment starting now, queued commitment, and the upfront payment due now.
    - (3) **Submit order** would buy the plan. **Do not choose it.**

    ![Cart](/images/3/3-4.png)

5. Select the plan (1), then choose **Remove from cart** (2):

    ![Remove from cart](/images/3/3-5.png)

6. (1) **Your cart is currently empty**, and (2) the total commitment is back to USD 0.00.

    ![Empty cart](/images/3/3-6.png)

{{% notice info %}}
After you submit an order, you can **delete a queued Savings Plan** (one with a future start date) before it starts. An active plan with an hourly commitment of USD 100 or less can be **returned** within 7 days of purchase, in the same calendar month. Otherwise, the commitment cannot be cancelled.
{{% /notice %}}
