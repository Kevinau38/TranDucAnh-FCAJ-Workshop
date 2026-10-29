---
title : "Basic DML (Data Manipulation Language)"
weight : 3
chapter : false
pre : " <b> 4.3. </b> "
---

#### Overview

**DML** (Data Manipulation Language) is the group of SQL statements that work with data in tables: `INSERT`, `SELECT`, `UPDATE`, and `DELETE`. In this step, you run SQL directly in the pgAdmin **Query Tool**.

{{% notice info %}}
This step uses `first_schema.first_table` from step 4.2.
{{% /notice %}}

#### Open the Query Tool

1. In **Object Explorer**:
    - (1) Right-click **first_database**.
    - (2) Choose **Query Tool**.

    ![Query Tool](/images/4-postgresql/4.3/1.png)

{{% notice tip %}}
You can also open the Query Tool with **Alt + Shift + Q**. The Query Tool connects to the database that you selected, shown in the bar above the editor.
{{% /notice %}}

#### Insert and query data

2. Insert the first row:
    - (1) Enter the following statement.
    - (2) Choose **Execute** (or press **F5**).

    ```sql
    INSERT INTO first_schema.first_table (col1) VALUES ('my first row');
    ```

    The **Messages** tab shows `INSERT 0 1`. The `1` is the number of inserted rows.

    ![INSERT](/images/4-postgresql/4.3/2.png)

{{% notice note %}}
pgAdmin runs with **auto-commit** on by default, so the new row is committed immediately.
{{% /notice %}}

3. Replace the statement and run it:
    - (1) Enter the following statement.
    - (2) Choose **Execute**.

    ```sql
    SELECT * FROM first_schema.first_table;
    ```

    ![SELECT](/images/4-postgresql/4.3/3.png)

#### Create a second table and return generated values

4. Replace the statement with the following block and run it:
    - (1) Enter the statements.
    - (2) Choose **Execute**.

    ```sql
    CREATE TABLE IF NOT EXISTS first_schema.second_table (
        col_pk        integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
        col_money     numeric(12,2),
        col_short_str varchar(100),
        col_datetime  timestamp(0)
    );

    INSERT INTO first_schema.second_table (col_money, col_short_str, col_datetime)
    VALUES (40.25, 'short string', TIMESTAMP '2026-10-29 10:23:54')
    RETURNING col_pk;
    ```

    `col_pk` is an **identity column**, so PostgreSQL generates its value. `RETURNING col_pk` returns the generated value, `1`, without another query.

    ![RETURNING](/images/4-postgresql/4.3/4.png)

{{% notice info %}}
`GENERATED ALWAYS AS IDENTITY` is the SQL-standard way to create an auto-increment column. Older PostgreSQL code often uses `serial`, which works in a similar way.
{{% /notice %}}

#### Convert data types

5. Replace the statement and run it:
    - (1) Enter the following statement.
    - (2) Choose **Execute**.

    ```sql
    SELECT '40.25' AS col1, 40.25 AS col2, '40.25'::numeric AS col3, CAST('40.25' AS numeric) AS col4;
    ```

    The **Data Output** shows the data type under each column name. `col1` is `text` because the value is quoted. `col3` and `col4` convert the text to `numeric` with `::` and `CAST`.

    ![Type conversion](/images/4-postgresql/4.3/5.png)

{{% notice note %}}
Unlike Oracle, PostgreSQL does not need a `FROM` clause in a `SELECT` statement, and it has no `dual` table.
{{% /notice %}}
