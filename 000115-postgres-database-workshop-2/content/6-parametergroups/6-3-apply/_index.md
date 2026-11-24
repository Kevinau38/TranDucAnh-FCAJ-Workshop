---
title : "Apply the parameter group"
weight : 3
chapter : false
pre : " <b> 6.3. </b> "
---

#### Overview

A DB instance uses only one DB parameter group at a time. When you change the group of a DB instance, RDS marks it **Pending reboot**: the new group, including its dynamic parameters, takes effect only after you reboot the DB instance. After that, changes to dynamic parameters in the same group apply without a reboot.

#### Attach the parameter group to rdspg-lab-db

1. Choose **Databases**, select `rdspg-lab-db`, then choose **Modify**. Scroll down to **Additional configuration** > **Database options**:
    - (1) **DB parameter group**: `rdspg-lab-pg18`

    ![DB parameter group](/images/6/6-3/1.png)

2. Choose **Continue**. On the summary page:
    - (1) Check the change: **DB parameter group** `default.postgres18` → `rdspg-lab-pg18`.
    - (2) Choose **Apply immediately**.
    - (3) Choose **Modify DB instance**.

    ![Summary of modifications](/images/6/6-3/2.png)

3. Choose `rdspg-lab-db` and open the **Configuration** tab:
    - (1) **DB instance parameter group** is `rdspg-lab-pg18` with the status **Pending reboot**.

    ![Pending reboot](/images/6/6-3/3.png)

#### Reboot the DB instance

4. Choose **Actions** > **Reboot**:
    - (1) Check that the DB instance is `rdspg-lab-db`.
    - (2) Choose **Confirm**. The database is unavailable for about one minute.

    ![Reboot](/images/6/6-3/4.png)

    When the **Status** is **Available** again, the parameter group status is **In sync**.

#### Check the new values

5. From the App server (`ssh rdspg-app`), run:

    ```bash
    psql -d pglab -c "SHOW log_connections;" \
                  -c "SHOW log_min_duration_statement;" \
                  -c "SELECT pg_sleep(5);"
    ```

    - (1) `log_connections` is `receipt,authentication,authorization`.
    - (2) `log_min_duration_statement` is `4s` (4000 ms).
    - (3) `pg_sleep(5)` runs for 5 seconds, longer than the 4-second limit, so PostgreSQL writes it to the log.

    ![SHOW](/images/6/6-3/5.png)

{{% notice info %}}
The PostgreSQL log now has lines such as `connection authorized: user=postgres database=pglab` and `duration: 5005.123 ms  statement: SELECT pg_sleep(5);`. Open them in the console (**Logs & events** > **Logs**) or, if you export the PostgreSQL log, in CloudWatch Logs (`/aws/rds/instance/rdspg-lab-db/postgresql`).
{{% /notice %}}

{{% notice warning %}}
`log_connections` and a low `log_min_duration_statement` can produce a lot of log data on a busy database. In production, choose a threshold that only catches the queries that you want to fix.
{{% /notice %}}

#### (Optional) AWS CLI

{{%expand "Attach the parameter group and reboot with the AWS CLI" %}}
```bash
aws rds modify-db-instance \
    --db-instance-identifier rdspg-lab-db \
    --db-parameter-group-name rdspg-lab-pg18 \
    --apply-immediately

aws rds reboot-db-instance --db-instance-identifier rdspg-lab-db

aws rds wait db-instance-available --db-instance-identifier rdspg-lab-db

aws rds describe-db-instances \
    --db-instance-identifier rdspg-lab-db \
    --query 'DBInstances[0].DBParameterGroups' --output table
```
{{% /expand %}}
