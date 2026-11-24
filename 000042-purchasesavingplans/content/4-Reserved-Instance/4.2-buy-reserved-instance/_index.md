---
title : "Purchase a Reserved Instance"
weight : 2
chapter : false
pre : " <b> 4.2. </b> "
---

#### Overview

You search for a Standard Reserved Instance for a `t3.micro` Linux instance in the Singapore Region, add it to the cart, and then delete it from the cart.

{{% notice warning %}}
This workshop never completes a purchase. Savings Plans, Reserved Instances, and reserved DB instances are commitments for 1 or 3 years: once the order is submitted, you pay for every hour of the term whether you use it or not. You stop at the cart, then empty it.
{{% /notice %}}

1. Open the [Amazon EC2 console](https://console.aws.amazon.com/ec2/home#ReservedInstances:) in the Region **Asia Pacific (Singapore)**:
    - (1) In the navigation pane, choose **Reserved Instances**. The list shows the RIs of the Region: active, queued, and retired.
    - (2) The banner reminds you that AWS recommends Savings Plans over Reserved Instances.
    - (3) Choose **Purchase Reserved Instances**.

    ![Reserved Instances](/images/4/4-1.png)

2. In **Purchase Reserved Instances**:
    - (1) **Platform**: Linux/UNIX, **Tenancy**: Default, **Offering class**: Standard, **Instance type**: `t3.micro`. Leave **Term** and **Payment option** as **Any**.
    - (2) Choose **Search**.
    - (3) The offerings: term, effective rate, upfront price, hourly rate, payment option, and normalized units per hour. The **Seller** column shows **AWS**, or a third party for offerings from the Reserved Instance Marketplace.
    - (4) For the **12 months, No upfront** offering, choose **Add to cart**.

    {{% notice tip %}}
    Turn on **Only show offerings that reserve capacity** to search for zonal RIs, which reserve capacity in an Availability Zone that you choose.
    {{% /notice %}}

    ![Search offerings](/images/4/4-2.png)

3. (1) The cart now has 1 Reserved Instance, with USD 0.00 due now and the average monthly recurring cost. (2) Choose **View cart**.

    ![Cart summary](/images/4/4-3.png)

4. In the **Shopping cart**:
    - (1) The item to order, its offering class, Region, prices, and **Order on** (**Now**, or a future date to queue the purchase).
    - (2) Choose **Delete** to remove the item from the cart.
    - (3) **Order all** would submit the order and charge any upfront fees immediately. **Do not choose it.**
    - (4) The note under the buttons confirms this.

    ![Shopping cart](/images/4/4-4.png)

5. The cart is empty. Close the dialog.

{{% notice info %}}
After purchase, an RI moves from **payment-pending** to **active**. A purchase with a future **Order on** date is **queued**; you can delete a queued RI before that date (**Actions** > **Delete queued Reserved Instances**). An active RI cannot be cancelled, but you can sell a Standard RI on the Reserved Instance Marketplace or exchange a Convertible RI.
{{% /notice %}}
