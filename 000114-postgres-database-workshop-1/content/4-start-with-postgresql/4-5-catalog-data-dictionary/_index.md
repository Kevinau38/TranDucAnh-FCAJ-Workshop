---
title : "Catalog and Data Dictionary"
weight : 5
chapter : false
pre : " <b> 4.5. </b> "
---

#### Overview

PostgreSQL stores information about its own objects, such as tables, columns, roles, and sessions, in system tables and views. Together they are called the **system catalog** or **data dictionary**.

| Source | Standard | Description |
|---|---|---|
| `pg_catalog` | PostgreSQL only | System tables and views, all named with the `pg_` prefix. Every database contains this schema |
| `information_schema` | SQL standard | Views that describe database objects in the same way across PostgreSQL, MySQL, SQL Server, and other databases |

#### Explore the catalog in pgAdmin

1. In **Object Explorer**, expand **first_database** > **Catalogs** > **PostgreSQL Catalog (pg_catalog)**. pgAdmin groups the catalog objects by type.

    ![PostgreSQL Catalog](/images/4-postgresql/4.5/1.png)

#### Query the catalog

Run each statement in the Query Tool of **first_database**: (1) enter the statement, (2) choose **Execute**.

2. List the system tables in `pg_catalog`:

    ```sql
    SELECT tablename FROM pg_tables WHERE schemaname = 'pg_catalog' ORDER BY tablename;
    ```

    ![pg_catalog tables](/images/4-postgresql/4.5/2.png)

3. Show the current sessions with the `pg_stat_activity` view:

    ```sql
    SELECT pid, usename, datname, application_name, state FROM pg_stat_activity WHERE datname IS NOT NULL;
    ```

    The result shows your pgAdmin sessions as `postgres`, and any other open sessions, such as `first_user` from `psql`. The `rdsadmin` sessions are internal connections that Amazon RDS uses to manage the instance.

    ![pg_stat_activity](/images/4-postgresql/4.5/3.png)

4. List the tables in `first_schema` with the SQL-standard `information_schema`:

    ```sql
    SELECT table_schema, table_name, table_type FROM information_schema.tables WHERE table_schema = 'first_schema';
    ```

    ![information_schema.tables](/images/4-postgresql/4.5/4.png)

{{% notice info %}}
`pg_stat_activity` belongs to the **cumulative statistics system**. Since PostgreSQL 15, these statistics are kept in shared memory instead of a separate statistics collector process.
{{% /notice %}}

{{% notice tip %}}
For performance diagnostics on Amazon RDS, use **CloudWatch Database Insights**, which you kept at the Standard level in step 2.7. It shows current and historical database load and wait events.
{{% /notice %}}
