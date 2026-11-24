---
title : "Manual snapshots"
weight : 3
chapter : false
pre : " <b> 4.3. </b> "
---

#### Overview

A **manual snapshot** is a backup that you take yourself, for example just before a release or a risky data change. RDS keeps it until you delete it, even after you delete the DB instance. A snapshot contains the whole DB instance, with all databases, not one database only.

#### Take a manual snapshot

1. In the [list of databases](https://console.aws.amazon.com/rds/home#databases:):
    - (1) Select `rdspg-lab-db`.
    - (2) Choose **Actions**.
    - (3) Choose **Take snapshot**.

    ![Take snapshot](/images/4/4-3/1.png)

2. On the **Take DB snapshot** page:
    - (1) **Snapshot type**: **DB instance**, `rdspg-lab-db`
    - (2) **Snapshot name**: `rdspg-lab-db-manual-snapshot`
    - (3) Choose **Take snapshot**.

    ![Snapshot name](/images/4/4-3/2.png)

The snapshot takes a few minutes. While it runs, the status of `rdspg-lab-db` is **Backing-up**, and the database keeps working normally. Wait until the status of the snapshot is **Available**.

{{% notice tip %}}
On a Single-AZ DB instance, the first seconds of a snapshot can briefly slow down I/O. On a Multi-AZ DB instance, RDS takes the snapshot from the standby, so the primary is not affected.
{{% /notice %}}

#### (Optional) AWS CLI

{{%expand "Take a manual snapshot with the AWS CLI" %}}
```bash
aws rds create-db-snapshot \
    --db-instance-identifier rdspg-lab-db \
    --db-snapshot-identifier rdspg-lab-db-manual-snapshot

aws rds wait db-snapshot-available \
    --db-snapshot-identifier rdspg-lab-db-manual-snapshot
```
{{% /expand %}}
