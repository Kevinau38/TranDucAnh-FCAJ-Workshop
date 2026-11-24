---
title : "Modify the backup settings"
weight : 2
chapter : false
pre : " <b> 4.2. </b> "
---

#### Overview

You increase the retention period to **3 days** and move the backup window to **22:00 UTC** (05:00 in Vietnam), a time with little traffic. A change of the backup settings does not restart the database.

#### Change the retention period and the backup window

1. Choose `rdspg-lab-db`, then **Modify**. In **Additional configuration** > **Backup**:
    - (1) **Backup retention period**: `3` days
    - (2) **Backup window**: **Choose a window**, **Start time** `22:00` UTC
    - (3) **Duration**: `0.5` hours

    ![Backup settings](/images/4/4-2/1.png)

2. Choose **Continue**:
    - (1) Check the changes: **Backup retention period** `1 day` → `3 days`, and the new **Backup window** `22:00-22:30`.
    - (2) Choose **Apply immediately**.
    - (3) Choose **Modify DB instance**.

    ![Summary of modifications](/images/4/4-2/2.png)

{{% notice info %}}
The retention period of automated backups is 0 to 35 days. A value of `0` turns off automated backups and point-in-time recovery, so do not use it in production. To keep a backup longer than 35 days, use a manual snapshot or AWS Backup.
{{% /notice %}}

#### (Optional) AWS CLI

{{%expand "Change the backup settings with the AWS CLI" %}}
```bash
aws rds modify-db-instance \
    --db-instance-identifier rdspg-lab-db \
    --backup-retention-period 3 \
    --preferred-backup-window 22:00-22:30 \
    --apply-immediately
```
{{% /expand %}}
