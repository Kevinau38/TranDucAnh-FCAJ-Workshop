---
title : "Restore a snapshot"
weight : 4
chapter : false
pre : " <b> 4.4. </b> "
---

#### Overview

You restore the manual snapshot from step 4.3 to a **new** DB instance, `rdspg-lab-db-restored`. The restored DB instance has the data of the moment when the snapshot was taken. `rdspg-lab-db` is not changed and keeps running.

{{% notice warning %}}
The restore page does not copy all settings from the source. By default, it can select a **Multi-AZ** deployment, a **Standard** instance class, and the **default** security group. Check every setting below, or the restored DB instance can cost much more and be unreachable from the App server.
{{% /notice %}}

#### Restore the snapshot

1. In the RDS console, choose **Snapshots**, then the **Manual** tab. Enter `rdspg` in the filter box:
    - (1) Select `rdspg-lab-db-manual-snapshot`.
    - (2) Choose **Actions**.
    - (3) Choose **Restore snapshot**.

    ![Restore snapshot](/images/4/4-4/1.png)

2. In **Availability and durability**, choose **Single-AZ DB instance deployment (1 instance)**. In **Settings**:
    - (1) **DB snapshot ID** is `rdspg-lab-db-manual-snapshot`.
    - (2) **DB instance identifier**: `rdspg-lab-db-restored`
    - (3) **DB instance class**: **Burstable classes**, then choose `db.t4g.micro` in **Instance type**.

    ![Restore settings](/images/4/4-4/2.png)

3. In **Connectivity**, keep `rdspg-lab-vpc` and:
    - (1) **DB subnet group**: `rdspg-lab-subnet-group`
    - (2) **Public access**: **No**
    - (3) **Existing VPC security groups**: remove `default` and choose `rds-sg` only.

    ![Connectivity](/images/4/4-4/3.png)

4. Keep the other settings, then choose **Restore DB instance** at the bottom of the page:
    - (1) RDS shows the message *Snapshot is being restored*.
    - (2) `rdspg-lab-db-restored` appears in the list with the status **Creating**.

    ![Restoring](/images/4/4-4/4.png)

5. After about 10 minutes:
    - (1) `rdspg-lab-db-restored` is **Available**, with the same engine and instance class as `rdspg-lab-db`.

    ![Restored DB instance](/images/4/4-4/5.png)

{{% notice tip %}}
The restored DB instance has its own endpoint. To check the data, connect from the App server with `psql -h <endpoint of rdspg-lab-db-restored> -d pglab`. The master password is the same as on `rdspg-lab-db`.
{{% /notice %}}

6. You do not need `rdspg-lab-db-restored` anymore. Select it, choose **Actions** > **Delete**, clear **Create final snapshot** and **Retain automated backups**, select the acknowledgment, type `delete me`, then choose **Delete**.

#### (Optional) AWS CLI

{{%expand "Restore and delete with the AWS CLI" %}}
```bash
aws rds restore-db-instance-from-db-snapshot \
    --db-snapshot-identifier rdspg-lab-db-manual-snapshot \
    --db-instance-identifier rdspg-lab-db-restored \
    --db-instance-class db.t4g.micro \
    --no-multi-az \
    --db-subnet-group-name rdspg-lab-subnet-group \
    --vpc-security-group-ids <rds-sg ID> \
    --no-publicly-accessible

aws rds wait db-instance-available --db-instance-identifier rdspg-lab-db-restored

aws rds delete-db-instance \
    --db-instance-identifier rdspg-lab-db-restored \
    --skip-final-snapshot \
    --delete-automated-backups
```
{{% /expand %}}
