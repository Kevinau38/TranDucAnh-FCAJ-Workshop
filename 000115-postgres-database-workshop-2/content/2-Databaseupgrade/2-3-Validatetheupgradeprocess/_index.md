---
title : "Validate the upgrade"
weight : 3
chapter : false
pre : " <b> 2.3. </b> "
---

#### Overview

After the major version upgrade, check the result in three places: the events of the DB instance, its configuration, and the database itself. Then refresh the optimizer statistics, because `pg_upgrade` does not copy them to the new version.

#### Check in the console

1. When the **Status** of `rdspg-lab-db` is **Available**, open the **Logs & events** tab:
    - (1) The event *Database instance engine major version upgrade complete. Previous version: 17.11.R1. New version: 18.6.R1* also shows the change of parameter group.

    ![Major version upgrade event](/images/2/2-3/1.png)

2. Open the **Configuration** tab:
    - (1) **Engine version** is **18.6**.
    - (2) **Option groups** is `default:postgres-18`, **In sync**.

    ![Configuration](/images/2/2-3/2.png)

#### Check from the App server

3. Connect to the App server (`ssh rdspg-app`), then run:

    ```bash
    psql -d postgres -c "SELECT version();"
    vacuumdb -d postgres --analyze-in-stages
    vacuumdb -d pglab --analyze-in-stages
    ```

    - (1) The server version is **PostgreSQL 18.6**.
    - (2) `vacuumdb --analyze-in-stages` rebuilds the optimizer statistics of each database, first quickly with few targets, then with full precision.

    ![Version and statistics](/images/2/2-3/3.png)

{{% notice warning %}}
Do not run `vacuumdb --all` on Amazon RDS. It also tries to connect to `rdsadmin`, an internal database that only AWS can use, and fails with *pg_hba.conf rejects connection*. Run `vacuumdb` for each of your databases instead.
{{% /notice %}}

{{% notice tip %}}
After a major version upgrade, also update the extensions you use with `ALTER EXTENSION ... UPDATE;`. RDS does not update them for you.
{{% /notice %}}

#### (Optional) AWS CLI

{{%expand "Check the engine version with the AWS CLI" %}}
```bash
aws rds describe-db-instances \
    --db-instance-identifier rdspg-lab-db \
    --query 'DBInstances[0].[EngineVersion,DBParameterGroups[0].DBParameterGroupName]' \
    --output table
```
{{% /expand %}}
