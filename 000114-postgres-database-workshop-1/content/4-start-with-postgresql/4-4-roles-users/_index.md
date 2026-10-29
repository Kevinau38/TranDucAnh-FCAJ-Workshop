---
title : "Roles and Users"
weight : 4
chapter : false
pre : " <b> 4.4. </b> "
---

#### Overview

In PostgreSQL, a **role** covers both users and groups. A **user** is simply a role with the `LOGIN` attribute. `CREATE USER` and `CREATE GROUP` are aliases of `CREATE ROLE`.

Roles are defined at the DB instance level and are valid in every database of the instance. Schemas and roles are separate concepts, unlike Oracle, where a schema is tied to a user.

In this step, you create:

| Role | Login | Privileges |
|---|---|---|
| `first_role` | No | Read `first_schema.first_table` only |
| `first_user` | Yes | Member of `first_role`, so it inherits the same privileges |

#### Explore existing roles

1. In the Query Tool of **first_database**, run:
    - (1) Enter the statement.
    - (2) Choose **Execute**.

    ```sql
    SELECT rolname, rolsuper, rolcanlogin FROM pg_roles
    WHERE rolname NOT LIKE 'pg\_%' ORDER BY rolname;
    ```

    ![Existing roles](/images/4-postgresql/4.4/1.png)

| Role | Description |
|---|---|
| `postgres` | The master user. `rolsuper` is `false` because Amazon RDS does not give you a real superuser |
| `rds_superuser` | The role that grants the master user its administrative privileges |
| `rdsadmin` | The real superuser that Amazon RDS uses to manage the instance |

#### Create a role and a user

2. Replace the statement with the following block and run it:
    - (1) Enter the statements.
    - (2) Choose **Execute**.

    ```sql
    CREATE ROLE first_role;
    GRANT USAGE ON SCHEMA first_schema TO first_role;
    GRANT SELECT ON first_schema.first_table TO first_role;

    CREATE ROLE first_user WITH LOGIN PASSWORD 'FirstUser#2026';
    GRANT first_role TO first_user;
    ```

    ![Create role and user](/images/4-postgresql/4.4/2.png)

{{% notice note %}}
A role needs two privileges to read a table in a schema: `USAGE` on the schema and `SELECT` on the table. Without `USAGE`, the role cannot access any object in the schema.
{{% /notice %}}

3. In **Object Explorer**, right-click **Login/Group Roles** and choose **Refresh**. `first_role` appears with a group icon, and `first_user` appears with a user icon because it can log in.

    ![Roles in Object Explorer](/images/4-postgresql/4.4/3.png)

#### Test the privileges

4. Connect to the App server (`ssh rdspg-app`) and log in as `first_user`. Enter `FirstUser#2026` when prompted.

    ```bash
    export RDSHOST="<RDS_ENDPOINT>"
    psql "host=$RDSHOST port=5432 dbname=first_database user=first_user sslmode=verify-full sslrootcert=./global-bundle.pem"
    ```

    Run each query on its own line:

    ```sql
    SELECT * FROM first_schema.first_table;
    ```

    ```sql
    SELECT * FROM first_schema.second_table;
    ```

    `first_user` can read `first_table`, but it gets `permission denied` for `second_table` because `first_role` has no privilege on that table.

    ![Test privileges](/images/4-postgresql/4.4/4.png)

5. Type `\q` to exit `psql`.

{{% notice tip %}}
Grant privileges to group roles, then add users to those roles. When a person changes jobs, you change their role membership instead of every table privilege. See [Managing PostgreSQL users and roles](https://aws.amazon.com/blogs/database/managing-postgresql-users-and-roles/) for best practices on Amazon RDS.
{{% /notice %}}
