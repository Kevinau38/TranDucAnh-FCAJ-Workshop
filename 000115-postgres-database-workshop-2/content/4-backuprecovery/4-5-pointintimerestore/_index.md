---
title : "Point-in-time recovery"
weight : 5
chapter : false
pre : " <b> 4.5. </b> "
---

#### Overview

A snapshot brings the database back to the moment when it was taken. Often you need a more exact moment, for example **one second before** someone ran a wrong `DROP TABLE` or `UPDATE` without `WHERE`. With **point-in-time recovery (PITR)**, RDS restores the last daily snapshot and then replays the transaction logs up to the second that you choose.

In this step, you simulate an accident: you create a table, drop it, then recover the table on a new DB instance restored to a moment between the two commands.

{{% notice info %}}
You can restore to any second between the earliest restorable time and the **latest restorable time**. The latest restorable time is usually less than 5 minutes behind the current time, because RDS uploads the transaction logs every 5 minutes.
{{% /notice %}}

#### Simulate the accident

1. On the App server, create a table with 1,000 rows:

    ```bash
    psql -d pglab -c "CREATE TABLE important_data AS SELECT g AS id, md5(g::text) AS payload, now() AS created_at FROM generate_series(1,1000) g;"
    ```

2. Wait a few seconds, then drop the table, and write down the time of the drop:

    ```bash
    psql -d pglab \
        -c "SELECT count(*), max(created_at) AS created_time FROM important_data;" \
        -c "SELECT now() AS drop_time;" \
        -c "DROP TABLE important_data;" \
        -c "SELECT count(*) FROM important_data;"
    ```

    - (1) The table has 1,000 rows, created at `01:23:22` UTC.
    - (2) The drop happens at `01:23:46` UTC.
    - (3) After the drop, the table does not exist anymore.

    ![The accident](/images/4/4-5/1.png)

    You will restore the database to `01:23:35` UTC, which is after the table was created and before it was dropped. Your times are different: choose a second between your `created_time` and your `drop_time`.

3. Wait about 5 minutes, then check the latest restorable time in **AWS CloudShell**:

    ```bash
    aws rds describe-db-instances \
        --db-instance-identifier rdspg-lab-db \
        --query 'DBInstances[0].{BackupRetentionPeriod:BackupRetentionPeriod,LatestRestorableTime:LatestRestorableTime}' \
        --output table
    ```

    - (1) The latest restorable time must be later than the time that you want to restore to.

    ![Latest restorable time](/images/4/4-5/2.png)

#### Restore to a point in time

4. In the [list of databases](https://console.aws.amazon.com/rds/home#databases:):
    - (1) Select `rdspg-lab-db`.
    - (2) Choose **Actions**.
    - (3) Choose **Restore to point in time**.

    ![Restore to point in time](/images/4/4-5/3.png)

5. In **Restore time**:
    - (1) Choose **Custom date and time**.
    - (2) **Date**: the date of the drop
    - (3) **Time**: the time that you chose in step 2. The console uses your local time zone, so convert from UTC. In this example, `01:23:35` UTC is `08:23:35` UTC+7.

    ![Restore time](/images/4/4-5/4.png)

6. In **Settings**:
    - (1) **Source DB instance identifier** is `rdspg-lab-db`.
    - (2) **DB instance identifier**: `rdspg-lab-db-pitr`

    ![Settings](/images/4/4-5/5.png)

7. In **Instance configuration** and **Availability and durability**:
    - (1) **Burstable classes**
    - (2) **Instance type**: `db.t4g.micro`
    - (3) **Single-AZ DB instance deployment (1 instance)**

    ![Instance configuration](/images/4/4-5/6.png)

8. In **Connectivity**:
    - (1) **DB subnet group**: `rdspg-lab-subnet-group`
    - (2) **Public access**: **No**
    - (3) **Existing VPC security groups**: `rds-sg` only

    Keep the other settings and choose **Restore to point in time** at the bottom of the page.

    ![Connectivity](/images/4/4-5/7.png)

9. Wait until the status of `rdspg-lab-db-pitr` is **Available**, about 10 to 15 minutes:
    - (1) `rdspg-lab-db-pitr` is **Available**.

    ![Restored DB instance](/images/4/4-5/8.png)

#### Check the recovered data

10. Open `rdspg-lab-db-pitr` and copy its **Endpoint**. On the App server, connect to the restored DB instance and count the rows. Enter the master password when `psql` asks for it:

    ```bash
    psql -h <endpoint of rdspg-lab-db-pitr> -d pglab \
        -c "SELECT count(*), max(created_at) AS created_time FROM important_data;"
    ```

    - (1) The table `important_data` exists again, with its 1,000 rows.

    ![Recovered data](/images/4/4-5/9.png)

In a real incident, you now copy the table back to the production database, for example with `pg_dump -t important_data` from `rdspg-lab-db-pitr` and `psql` into `rdspg-lab-db`, or you switch the application to the restored DB instance.

11. Delete `rdspg-lab-db-pitr`: select it, choose **Actions** > **Delete**, clear **Create final snapshot** and **Retain automated backups**, select the acknowledgment, type `delete me`, then choose **Delete**.

#### (Optional) AWS CLI

{{%expand "Restore to a point in time with the AWS CLI" %}}
```bash
aws rds restore-db-instance-to-point-in-time \
    --source-db-instance-identifier rdspg-lab-db \
    --target-db-instance-identifier rdspg-lab-db-pitr \
    --restore-time 2026-11-11T01:23:35Z \
    --db-instance-class db.t4g.micro \
    --no-multi-az \
    --db-subnet-group-name rdspg-lab-subnet-group \
    --vpc-security-group-ids <rds-sg ID> \
    --no-publicly-accessible

aws rds wait db-instance-available --db-instance-identifier rdspg-lab-db-pitr

aws rds delete-db-instance \
    --db-instance-identifier rdspg-lab-db-pitr \
    --skip-final-snapshot \
    --delete-automated-backups
```
Use `--use-latest-restorable-time` instead of `--restore-time` to restore to the latest restorable time.
{{% /expand %}}
