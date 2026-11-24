---
title : "Promote the read replica"
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### Overview

**Promoting** a read replica stops the replication and turns the replica into a normal, standalone DB instance that accepts writes. You use it, for example:

- to recover when the primary of a Single-AZ deployment is lost,
- to move a database to another Region (promote a cross-Region replica),
- to split a database, or to create a test copy with recent data.

{{% notice warning %}}
Promotion cannot be undone. The promoted DB instance does not receive changes from the old primary anymore. Before you promote, stop the writes to the primary and wait until the replica lag is close to zero, otherwise the last transactions are lost.
{{% /notice %}}

#### Promote the replica

1. Choose `rdspg-lab-db-read`:
    - (1) Select the replica in **Related**.
    - (2) Choose **Actions**.
    - (3) Choose **Promote**.

    ![Promote](/images/5/5-2/1.png)

2. On the **Promote read replica** page:
    - (1) Keep **Enable automated backup** selected. A replica has no backups of its own; after promotion it needs them like any other DB instance.
    - (2) **Backup retention period**: `1` day
    - (3) Choose **Promote read replica**.

    ![Promote settings](/images/5/5-2/2.png)

3. Both DB instances go to the status **Modifying** for a few minutes. RDS restarts the replica in read-write mode and takes a first backup:
    - (1) `rdspg-lab-db` and `rdspg-lab-db-read` are now two separate DB instances with the role **Instance**. The promoted instance stays in **Backing-up** for a few more minutes.

    ![Standalone instances](/images/5/5-2/3.png)

4. You do not need `rdspg-lab-db-read` anymore. Select it, choose **Actions** > **Delete**, clear **Create final snapshot** and **Retain automated backups**, select the acknowledgment, type `delete me`, then choose **Delete**.

#### (Optional) AWS CLI

{{%expand "Promote and delete with the AWS CLI" %}}
```bash
aws rds promote-read-replica \
    --db-instance-identifier rdspg-lab-db-read \
    --backup-retention-period 1

aws rds wait db-instance-available --db-instance-identifier rdspg-lab-db-read

aws rds delete-db-instance \
    --db-instance-identifier rdspg-lab-db-read \
    --skip-final-snapshot \
    --delete-automated-backups
```
{{% /expand %}}
