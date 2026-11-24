---
title : "Create a read replica"
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Overview

A **read replica** is a read-only copy of a DB instance. RDS creates it from a snapshot of the source, then keeps it up to date with the **asynchronous streaming replication** of PostgreSQL. Applications can send read-only queries (reports, dashboards, searches) to the replica and keep the primary free for writes.

| | Primary | Read replica |
|---|---|---|
| Reads | Yes | Yes |
| Writes | Yes | No, the replica is read-only |
| Endpoint | Its own | Its own, different from the primary |
| Data | Current | A few seconds behind (**replica lag**) |
| Number | 1 | Up to 15 per source, in the same or another Region |

#### Create the read replica

1. In the [list of databases](https://console.aws.amazon.com/rds/home#databases:):
    - (1) Select `rdspg-lab-db`.
    - (2) Choose **Actions**.
    - (3) Choose **Create read replica**.

    ![Create read replica](/images/5/5-1/1.png)

2. In **Settings** and **Instance configuration**:
    - (1) **Replica source** is `rdspg-lab-db`.
    - (2) **DB instance identifier**: `rdspg-lab-db-read`
    - (3) **Burstable classes**, **Instance type** `db.t4g.micro`

    ![Settings](/images/5/5-1/2.png)

{{% notice tip %}}
A replica can use a different instance class than its primary. Keep the same class or larger for a replica that you might promote, or one that must keep up with a busy primary.
{{% /notice %}}

3. In **Availability**, the console selects **Multi-AZ DB instance deployment** by default:
    - (1) Choose **Single-AZ DB instance deployment (1 instance)** to keep the cost low.

    ![Availability](/images/5/5-1/3.png)

4. In **Connectivity**, the console copies the settings of the source. Check them:
    - (1) **DB subnet group**: `rdspg-lab-subnet-group`
    - (2) **Public access**: **Not publicly accessible**
    - (3) **Existing VPC security groups**: `rds-sg`

    ![Connectivity](/images/5/5-1/4.png)

    Keep the other settings and choose **Create read replica** at the bottom of the page.

5. In the list of databases:
    - (1) The status of `rdspg-lab-db` changes to **Modifying** while RDS takes a snapshot of it. Its role is now **Primary**.
    - (2) `rdspg-lab-db-read` appears under it with the role **Replica** and the status **Creating**.

    ![Creating](/images/5/5-1/5.png)

    The replica takes about 10 to 15 minutes. `rdspg-lab-db` stays available the whole time.

#### Review the replica

6. When `rdspg-lab-db-read` is **Available**, choose it. On the **Connectivity & security** tab, choose **Endpoints** in **Connect using** and expand **Additional configurations**:
    - (1) The replica has its own **Endpoint**, different from the endpoint of `rdspg-lab-db`.
    - (2) It uses the same security group `rds-sg`.

    ![Replica endpoint](/images/5/5-1/6.png)

7. Scroll down to **Replication**:
    - (1) `rdspg-lab-db-read` has the role **Replica**, the source `rdspg-lab-db`, the state **Replicating**, and a lag of a few seconds.

    ![Replication](/images/5/5-1/7.png)

#### Connect to the replica

8. On the App server, connect to the endpoint of the replica. Enter the master password when `psql` asks for it:

    ```bash
    R=<endpoint of rdspg-lab-db-read>
    psql -h $R -d pglab \
        -c "SELECT pg_is_in_recovery() AS is_replica, count(*) AS accounts FROM pgbench_accounts;" \
        -c "SELECT extract(epoch from now() - pg_last_xact_replay_timestamp()) AS replica_lag_seconds;" \
        -c "CREATE TABLE test_write(id int);"
    ```

    - (1) `pg_is_in_recovery()` is `t`: the server is a replica. It has all 5,000,000 rows of `pgbench_accounts` from chapter 3.
    - (2) `replica_lag_seconds` is the time since the last transaction that the replica applied.
    - (3) The replica rejects writes: *cannot execute CREATE TABLE in a read-only transaction*.

    ![psql on the replica](/images/5/5-1/8.png)

{{% notice note %}}
`replica_lag_seconds` grows when nobody writes to the primary, because it measures the time since the **last** replayed transaction. It is a good lag indicator only while the primary is busy. The CloudWatch metric `ReplicaLag` of the replica is more reliable.
{{% /notice %}}

#### (Optional) AWS CLI

{{%expand "Create the read replica with the AWS CLI" %}}
```bash
aws rds create-db-instance-read-replica \
    --db-instance-identifier rdspg-lab-db-read \
    --source-db-instance-identifier rdspg-lab-db \
    --db-instance-class db.t4g.micro \
    --no-multi-az \
    --no-publicly-accessible

aws rds wait db-instance-available --db-instance-identifier rdspg-lab-db-read
```
{{% /expand %}}
