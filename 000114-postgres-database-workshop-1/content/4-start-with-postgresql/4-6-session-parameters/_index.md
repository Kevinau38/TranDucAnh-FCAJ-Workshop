---
title : "Session Parameters"
weight : 6
chapter : false
pre : " <b> 4.6. </b> "
---

#### Overview

PostgreSQL behavior is controlled by **parameters**. Some parameters apply to the whole DB instance, and some can be changed for a single **session** (one connection).

| Scope | How to change | Takes effect |
|---|---|---|
| Whole DB instance | Edit the **DB parameter group** in Amazon RDS, for example `default.postgres18` from step 2.7 | Immediately or after a reboot, depending on the parameter |
| One session | Run `SET` in the session | Immediately, only for the current connection |

#### View parameters

Run each statement in the Query Tool of **first_database**: (1) enter the statement, (2) choose **Execute**.

1. List the parameters that a user can change for a session:

    ```sql
    SELECT name, setting, context FROM pg_settings WHERE context = 'user' ORDER BY name;
    ```

    ![User-level parameters](/images/4-postgresql/4.6/1.png)

2. Show the current value of `search_path`:

    ```sql
    SHOW search_path;
    ```

    The default value is `"$user", public`. PostgreSQL first looks for a schema with the same name as the current user, then in `public`.

    ![SHOW search_path](/images/4-postgresql/4.6/2.png)

#### Change a session parameter

3. Query `first_table` without the schema name:

    ```sql
    SELECT * FROM first_table;
    ```

    The query fails with `relation "first_table" does not exist` because `first_schema` is not in `search_path`.

    ![Table not found](/images/4-postgresql/4.6/3.png)

4. Add `first_schema` to `search_path` for this session, then run the query again:

    ```sql
    SET search_path TO first_schema, public;
    SELECT * FROM first_table;
    ```

    The query now returns `my first row`.

    ![SET search_path](/images/4-postgresql/4.6/4.png)

{{% notice note %}}
`SET` changes the value only for the current session. Other connections, and new connections, still use the default value. Run `RESET search_path;` to return to the default value in the current session.
{{% /notice %}}

{{% notice tip %}}
After you change a parameter in a DB parameter group, confirm the value that is in effect with `SHOW` or `pg_settings`. Some parameters change only after the DB instance reboots, so the value in the parameter group and the value in effect can be different.
{{% /notice %}}
