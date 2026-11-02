---
title : "Databases and Schemas"
weight : 1
chapter : false
pre : " <b> 4.1. </b> "
---

#### Overview

PostgreSQL organizes objects in four levels:

```
DB instance (rdspg-lab-db) > Database > Schema > Table
```

| Level | Description |
|---|---|
| DB instance | Also called a PostgreSQL **cluster**. Roles (users) are shared across all databases in the instance |
| Database | Data is not shared between databases. A connection works with only one database at a time |
| Schema | A namespace inside a database, similar to a folder. Two schemas can contain tables with the same name |

Every database has a default schema named `public`. Unlike MySQL, where a database and a schema are almost the same thing, PostgreSQL keeps them as separate levels.

{{% notice info %}}
This chapter assumes that pgAdmin 4 is connected to `rdspg-lab-db` as in step 3.2.
{{% /notice %}}

#### Create a database

1. In **Object Explorer**, expand **rdspg-lab-db**:
    - (1) Right-click **Databases**.
    - (2) Choose **Create**.
    - (3) Choose **Database...**.

    ![Create database](/images/4-postgresql/4.1/1.png)

2. In the **General** tab:
    - (1) **Database**: `first_database`. Keep **Owner** as `postgres`.

    ![Database name](/images/4-postgresql/4.1/2.png)

3. Review the statement that pgAdmin runs:
    - (1) Choose the **SQL** tab.
    - (2) Choose **Save**.

    ![CREATE DATABASE statement](/images/4-postgresql/4.1/3.png)

{{% notice note %}}
Every action in pgAdmin runs a SQL statement. The **SQL** tab shows that statement before you save, so you can learn the SQL behind each action.
{{% /notice %}}

4. Expand **first_database** > **Schemas**. The new database already contains the `public` schema.

    ![first_database and public schema](/images/4-postgresql/4.1/4.png)

#### Create a schema

5. Under **first_database**:
    - (1) Right-click **Schemas**.
    - (2) Choose **Create**.
    - (3) Choose **Schema...**.

    ![Create schema](/images/4-postgresql/4.1/5.png)

6. In the **General** tab:
    - (1) **Name**: `first_schema`. Keep **Owner** as `postgres`.
    - (2) Choose **Save**.

    ![Schema name](/images/4-postgresql/4.1/6.png)

7. `first_schema` appears under **Schemas**.

    ![first_schema](/images/4-postgresql/4.1/7.png)

{{% notice tip %}}
When a query uses a table name without a schema, PostgreSQL looks for it in the schemas listed in `search_path`, which is `"$user", public` by default. A table in `first_schema` must be written as `first_schema.table_name` unless you change `search_path`. You practice this in step 4.6.
{{% /notice %}}
