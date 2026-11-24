---
title : "Reserved DB instances"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

#### Overview

A **reserved DB instance** is a billing discount for Amazon RDS or Amazon Aurora, for 1 or 3 years. Like an EC2 Reserved Instance, it is not a database: you create DB instances as usual, and the ones that match the reservation are billed at the discounted rate. The discount covers only the instance hours, not storage, backups, or I/O.

To receive the discount, a DB instance must match the reservation on:

- AWS Region
- DB engine (the engine version does not matter)
- DB instance class, for example `db.t4g.micro`
- License model (license-included or bring-your-own-license)

**Size flexibility**: for RDS for PostgreSQL, MySQL, MariaDB, Db2, and Oracle BYOL, the discount applies to any size in the same instance class family, and to both Single-AZ and Multi-AZ deployments, counted in **normalized units** (for example, `micro` = 0.5 unit, `small` = 1, `large` = 4; Multi-AZ counts double). RDS for SQL Server and Oracle license-included need an exact match.

| Offering type | Terms available |
|---|---|
| **No Upfront** | 1 year only |
| **Partial Upfront** | 1 or 3 years |
| **All Upfront** | 1 or 3 years |

{{% notice warning %}}
This workshop never completes a purchase. Savings Plans, Reserved Instances, and reserved DB instances are commitments for 1 or 3 years: once the order is submitted, you pay for every hour of the term whether you use it or not. You stop at the cart, then empty it.
{{% /notice %}}

#### Explore the purchase page

1. Open the [Amazon RDS console](https://console.aws.amazon.com/rds/home#reserved-instances:) in the Region **Asia Pacific (Singapore)**:
    - (1) In the navigation pane, choose **Reserved instances**.
    - (2) The list of reservations in the Region, empty here.
    - (3) Choose **Purchase Reserved DB Instance**.

    ![Reserved instances](/images/5/5-1.png)

2. Open **Engine Type** (1). The list (2) shows every engine that supports reservations: Aurora MySQL and Aurora PostgreSQL, RDS for MySQL, MariaDB, and PostgreSQL, Oracle, SQL Server, Db2, and RDS Custom. Choose **RDS PostgreSQL**.

    ![Engine type](/images/5/5-2.png)

3. Fill in the reservation:
    - (1) **Engine Type**: RDS PostgreSQL.
    - (2) **DB instance class**: type `db.t4g.micro` in the search box and choose it.
    - (3) **Deployment options**: **Single-AZ DB instance**.
    - (4) **Term**: **1 year**.
    - (5) **Offering type**: **No Upfront**.

    ![Reserved DB instance options](/images/5/5-3.png)

4. Scroll down:
    - (1) **Number of DB instances**: 1.
    - (2) **Reserved (ID) - optional**: a name to track the reservation. Leave it empty.
    - (3) Choose **Next**. This only opens the review page.

    ![Number of DB instances](/images/5/5-4.png)

5. The **Review and purchase** page:
    - (1) The **Purchase summary**: normalized units per hour (0.5 for a micro instance), the hourly usage charge, the engine, the instance class, and the term starting today.
    - (2) **Total one-time payment** is USD 0.00 for No Upfront; the hourly charges appear on your monthly bill.
    - (3) RDS warns that no DB instance in your account matches the reservation: you are charged anyway for every hour of the term.
    - (4) Choose **Cancel**.
    - (5) **Purchase reserved instance** would submit the order. **Do not choose it.**

    ![Review and purchase](/images/5/5-5.png)

{{% notice info %}}
A reserved DB instance cannot be cancelled. If you delete the DB instance that uses the reservation, you keep paying for the reservation, and the discount moves to any other matching DB instance that you create in the same Region.
{{% /notice %}}

#### Summary

| | Savings Plans | EC2 Reserved Instances | Reserved DB instances |
|---|---|---|---|
| You commit to | USD per hour | An instance configuration | A DB engine and instance class |
| Best for | Most compute and database usage | Capacity reservation in an Availability Zone | Steady RDS and Aurora instances |
| Change after purchase | Not needed, applies automatically | Modify, exchange (Convertible), or sell (Standard) | Not possible |

You did not buy anything in this workshop, so there is nothing to clean up.
