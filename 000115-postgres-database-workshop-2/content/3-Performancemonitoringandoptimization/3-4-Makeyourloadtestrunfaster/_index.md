---
title : "Make the load test run faster"
weight : 4
chapter : false
pre : " <b> 3.4. </b> "
---

#### Overview

Database Insights showed that the load test is limited by CPU. To remove this bottleneck, you **scale up** the DB instance, which is called *vertical scaling*: you change the instance class to one with more vCPUs and memory, run the same load test again, and then scale back down.

| Instance class | vCPUs | Memory |
|---|---|---|
| `db.t4g.micro` | 2 | 1 GiB |
| `db.t4g.xlarge` | 4 | 16 GiB |

`db.t4g.xlarge` is not in the free tier and is charged per hour, so keep it only for the duration of the test.

{{% notice warning %}}
Changing the instance class restarts the database. With **Apply immediately**, the database is unavailable for a few minutes. In production, schedule this change in the maintenance window, or use a Multi-AZ deployment to shorten the downtime.
{{% /notice %}}

#### Scale up the DB instance

1. Choose `rdspg-lab-db`, then **Modify**. In **Instance configuration**:
    - (1) **Instance type**: `db.t4g.xlarge` (4 vCPUs, 16 GiB RAM).

    ![Instance type](/images/3/3-4/1.png)

2. Choose **Continue**:
    - (1) Check the change: **DB instance class** `db.t4g.micro` → `db.t4g.xlarge`.
    - (2) Choose **Apply immediately**.
    - (3) Choose **Modify DB instance**.

    ![Summary of modifications](/images/3/3-4/2.png)

3. Wait about 10 minutes until the **Size** column shows `db.t4g.xlarge` and the **Status** is **Available**.

#### Run the load test again

4. On the App server, run exactly the same command as in step 3.2:

    ```bash
    pgbench --protocol=prepared -P 30 --time=300 --client=50 --jobs=2 pglab
    ```

    - (1) The average latency drops from about 148 ms to about **18 ms**.
    - (2) The throughput rises from about 337 to about **2,780 tps**, about 8 times faster.

    ![Load test on db.t4g.xlarge](/images/3/3-4/3.png)

The improvement is bigger than the number of vCPUs alone suggests. With 16 GiB of memory, the whole test data (about 750 MB) fits in the shared buffers, so PostgreSQL reads almost nothing from storage. And `db.t4g.xlarge` starts with more CPU credits than `db.t4g.micro`.

#### Scale back down

5. To keep the cost low, modify `rdspg-lab-db` again and change the instance type back to `db.t4g.micro`, with **Apply immediately**.

{{% notice tip %}}
Scaling up is the fastest way to remove a CPU or memory bottleneck, but it has a limit and costs more every hour. Before you scale up, check with Database Insights that the bottleneck really is the instance, and not a missing index or a badly written query.
{{% /notice %}}

#### (Optional) AWS CLI

{{%expand "Change the instance class with the AWS CLI" %}}
```bash
aws rds modify-db-instance \
    --db-instance-identifier rdspg-lab-db \
    --db-instance-class db.t4g.xlarge \
    --apply-immediately

# After the load test
aws rds modify-db-instance \
    --db-instance-identifier rdspg-lab-db \
    --db-instance-class db.t4g.micro \
    --apply-immediately
```
{{% /expand %}}
