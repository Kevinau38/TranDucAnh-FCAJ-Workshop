---
title : "Back up and restore the database"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

Multi-AZ protects the database against the failure of one Availability Zone, but it does not protect you from a wrong change: a deleted post is deleted on the standby too. A **DB snapshot** keeps a point-in-time copy that you can restore into a new DB instance.

#### Create a snapshot

1. In **AWS CloudShell**, run:

   ```bash
   aws rds create-db-snapshot --db-instance-identifier wordpress-db --db-snapshot-identifier wordpress-db-snapshot --query "DBSnapshot.[DBSnapshotIdentifier,Status]" --output text
   ```

2. Open the [Amazon RDS console](https://console.aws.amazon.com/rds/) → **Snapshots** → **Manual**, and (1) filter by `wordpress-db`. After a few minutes the snapshot is (2) **Available** and the progress is **Completed**.

   ![DB snapshot](/images/5-backup/5.1-snapshot.png)

#### Restore the snapshot into a new instance

A restore always creates a **new** DB instance. You restore it into the same private subnet group and `Database-SG`. To save cost, the restored copy is Single-AZ.

1. In **AWS CloudShell**, run:

   ```bash
   VPC=$(aws ec2 describe-vpcs --filters Name=tag:Name,Values=wordpress-vpc --query "Vpcs[0].VpcId" --output text)
   DBSG=$(aws ec2 describe-security-groups --filters Name=vpc-id,Values=$VPC Name=group-name,Values=Database-SG --query "SecurityGroups[0].GroupId" --output text)
   aws rds restore-db-instance-from-db-snapshot --db-instance-identifier wordpress-db-restored \
     --db-snapshot-identifier wordpress-db-snapshot --db-instance-class db.t4g.micro \
     --db-subnet-group-name wordpress-db-subnet-group --vpc-security-group-ids $DBSG \
     --no-multi-az --no-publicly-accessible --no-deletion-protection \
     --query "DBInstance.[DBInstanceIdentifier,DBInstanceStatus,MultiAZ]" --output text
   ```

2. Open **Databases**. After about 10 minutes (1) `wordpress-db-restored` is **Available** next to the original `wordpress-db`, in the same private DB subnet group.

   ![Restored instance](/images/5-backup/5.2-restored.png)

{{% notice tip %}}
To move WordPress to the restored database, you would change `DB_HOST` in `wp-config.php` to the endpoint of `wordpress-db-restored`, create a new AMI and update the launch template. In this lab you only verify the restore, and you delete the restored instance in the clean-up step.
{{% /notice %}}
