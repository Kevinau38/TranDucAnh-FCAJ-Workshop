---
title : "Automated backups"
weight : 1
chapter : false
pre : " <b> 4.1. </b> "
---

#### Overview

RDS takes a full snapshot of the storage volume right after the DB instance is created, then one snapshot per day in the **backup window**. Between the snapshots, RDS uploads the transaction logs (WAL) to Amazon S3 every 5 minutes. Together, they let you restore the database to any second between the **earliest** and the **latest restorable time**.

#### Review the automated backups

1. In the [Amazon RDS console](https://console.aws.amazon.com/rds/home#databases:), choose `rdspg-lab-db`, then open the **Maintenance & backups** tab:
    - (1) **Automated backups** is enabled with a retention period of **1 day**, the value you chose when you created the database.
    - (2) **Latest restore time** is usually less than 5 minutes ago.
    - (3) The **Backup window** is the 30-minute period of the daily backup, in UTC. RDS chose it at random because you did not set one.
    - (4) **Snapshots** lists the automated snapshots: the first one taken after the database was created, the snapshots taken by the minor and major version upgrades, and `rds:preupgrade-...`, the snapshot that RDS took just before the major version upgrade in step 2.2.

    ![Maintenance and backups](/images/4/4-1/1.png)

#### (Optional) Check the backups with the AWS CLI

2. Open **AWS CloudShell** and run:

    ```bash
    aws rds describe-db-instances \
        --db-instance-identifier rdspg-lab-db \
        --query 'DBInstances[0].[BackupRetentionPeriod,PreferredBackupWindow,LatestRestorableTime]' \
        --output table

    aws rds describe-db-snapshots \
        --db-instance-identifier rdspg-lab-db \
        --query 'DBSnapshots[].[DBSnapshotIdentifier,SnapshotType,Status]' \
        --output table
    ```

    - (1) The retention period (`1`), the backup window, and the latest restorable time in UTC are the same as in the console.
    - (2) All snapshots have the type `automated`. A snapshot that you take yourself has the type `manual`.

    ![AWS CLI](/images/4/4-1/2.png)

{{% notice note %}}
You cannot delete an automated snapshot yourself. RDS deletes it when it is older than the retention period. When you delete the DB instance, you can choose to keep the automated backups for the rest of the retention period.
{{% /notice %}}
