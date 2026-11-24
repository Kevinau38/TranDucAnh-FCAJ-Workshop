---
title : "Set up Multi-AZ"
weight : 1
chapter : false
pre : " <b> 7.1. </b> "
---

#### Overview

When you convert a Single-AZ DB instance to Multi-AZ, RDS takes a snapshot of the primary, restores it as a standby in another Availability Zone, and then starts synchronous replication. The primary stays available during the conversion, but its performance can drop while the standby catches up.

#### Check the current setting

1. Choose **Databases** and scroll the list to the right:
    - (1) `rdspg-lab-db` is a `db.t4g.micro` DB instance.
    - (2) The **Multi-AZ** column shows **No**.
    - (3) Select `rdspg-lab-db` and choose **Modify**.

    ![Multi-AZ No](/images/7/7-1/1.png)

#### Convert to Multi-AZ

2. On the **Modify DB instance: rdspg-lab-db** page, scroll down to **Availability & durability**:
    - (1) **Multi-AZ deployment**: **Create a standby instance (recommended for production usage)**

    ![Create a standby instance](/images/7/7-1/2.png)

3. Choose **Continue**. On the summary page:
    - (1) Check the change: **Multi-AZ deployment** No → Yes.
    - (2) Choose **Apply immediately**.
    - Read the warning: the conversion can have a performance impact. On a busy production database, you can instead create a read replica, convert the replica to Multi-AZ, and promote it.
    - (3) Choose **Modify DB instance**.

    ![Summary of modifications](/images/7/7-1/3.png)

4. The **Status** of `rdspg-lab-db` changes to **Modifying**. When it is **Available** again (about 10 to 20 minutes), choose `rdspg-lab-db` and open the **Configuration** tab:
    - (1) **Multi-AZ** is **Yes**.
    - (2) **Storage operation status** can show **Initializing** for a while: RDS is still loading the storage of the standby in the background. The DB instance is available, but its latency can be higher until this finishes.

    ![Multi-AZ Yes](/images/7/7-1/4.png)

#### (Optional) AWS CLI

{{%expand "Convert to Multi-AZ with the AWS CLI" %}}
```bash
aws rds modify-db-instance \
    --db-instance-identifier rdspg-lab-db \
    --multi-az \
    --apply-immediately

aws rds wait db-instance-available --db-instance-identifier rdspg-lab-db

aws rds describe-db-instances \
    --db-instance-identifier rdspg-lab-db \
    --query 'DBInstances[0].[MultiAZ,AvailabilityZone,SecondaryAvailabilityZone]' \
    --output table
```
{{% /expand %}}
