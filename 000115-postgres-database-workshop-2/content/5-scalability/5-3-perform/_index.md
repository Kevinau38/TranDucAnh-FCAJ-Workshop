---
title : "Vertical scaling"
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

#### Overview

You already scaled `rdspg-lab-db` vertically in step [3.4](../../3-performancemonitoringandoptimization/3-4-makeyourloadtestrunfaster/): you changed the instance class from `db.t4g.micro` to `db.t4g.xlarge`, the throughput of the load test rose from about 337 to about 2,780 transactions per second, and you changed the class back.

This page summarizes what you need to know before you scale up or down in production.

| Change | Downtime | How to reduce it |
|---|---|---|
| Instance class (CPU, memory) | A few minutes, the DB instance restarts | Multi-AZ: RDS changes the standby first, then fails over (about 1 minute) |
| Allocated storage (increase) | None | Storage autoscaling grows the volume automatically |
| IOPS and throughput (`gp3`, `io2`) | None | — |
| Storage type (`gp2` → `gp3`) | None, but performance can drop during the change | — |

{{% notice note %}}
You can increase the allocated storage, but you cannot decrease it. After a storage change, you must wait at least 6 hours, or until the storage optimization finishes, before the next storage change.
{{% /notice %}}

#### Steps

1. Choose the DB instance, then **Modify**.
2. In **Instance configuration**, choose the new **Instance type**. In **Storage**, change the allocated storage or the IOPS if needed.
3. Choose **Continue**, check the **Summary of modifications**, then choose **Apply immediately** or **Apply during the next scheduled maintenance window**.
4. Choose **Modify DB instance**, and wait until the status is **Available** again.

{{% notice tip %}}
Watch CloudWatch Database Insights for a few days before you choose a smaller instance class. Scale down only when the CPU utilization and the database load stay low, and the database fits in memory.
{{% /notice %}}

#### (Optional) AWS CLI

{{%expand "Change the instance class and the storage with the AWS CLI" %}}
```bash
aws rds modify-db-instance \
    --db-instance-identifier rdspg-lab-db \
    --db-instance-class db.t4g.small \
    --allocated-storage 30 \
    --max-allocated-storage 100 \
    --apply-immediately
```
`--max-allocated-storage` turns on storage autoscaling up to 100 GiB.
{{% /expand %}}
