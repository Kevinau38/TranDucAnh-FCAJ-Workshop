---
title : "Tables and Data Types"
weight : 2
chapter : false
pre : " <b> 4.2. </b> "
---

#### Overview

A **table** is a named collection of rows. Every row has the same columns, and each column has a specific **data type**. PostgreSQL documentation also calls a table a **relation**, the mathematical term behind "relational database".

SQL does not guarantee the order of rows in a table. To get rows in a specific order, use `ORDER BY` in your query.

#### Create a table

1. In **Object Explorer**, under **first_database** > **Schemas**:
    - (1) Right-click **first_schema**.
    - (2) Choose **Create**.
    - (3) Choose **Table...**.

    ![Create table](/images/4-postgresql/4.2/1.png)

2. In the **General** tab:
    - (1) **Name**: `first_table`
    - (2) **Schema**: `first_schema`

    ![Table name](/images/4-postgresql/4.2/2.png)

3. In the **Columns** tab:
    - (1) Choose the **Columns** tab.
    - (2) Choose **+** to add a column.
    - (3) **Name**: `col1`
    - (4) **Data type**: `text`
    - (5) Turn on **Not NULL?**.
    - (6) Turn on **Primary key?**.

    ![Columns](/images/4-postgresql/4.2/3.png)

4. Review the statement and save:
    - (1) Choose the **SQL** tab. pgAdmin creates the table with `CREATE TABLE first_schema.first_table`.
    - (2) Choose **Save**.

    ![CREATE TABLE statement](/images/4-postgresql/4.2/4.png)

5. Expand **first_table** > **Columns** to see `col1` with the `text` data type.

    ![first_table](/images/4-postgresql/4.2/5.png)

{{% notice note %}}
The **General** tab also has **Partitioned table?**, and the **Columns** tab has **Inherited from table(s)**. This workshop does not use declarative partitioning or table inheritance, but both are useful PostgreSQL features for large or related tables.
{{% /notice %}}

#### Common data types

| Group | Data type | When to use |
|---|---|---|
| Whole numbers | `integer`, `bigint` | `integer` is the common choice. Use `bigint` for very large values |
| Exact decimals | `numeric(p, s)` | Money and other values that must be exact |
| Auto-increment | `serial`, `bigserial`, or `GENERATED ... AS IDENTITY` | Unique ID columns. Identity columns are the SQL-standard option |
| Text | `text`, `varchar(n)` | `text` has no length limit. `varchar(n)` limits the length. Both have the same performance |
| Date and time | `date`, `time`, `timestamp`, `timestamptz` | `timestamptz` stores a timestamp with time zone |
| Boolean | `boolean` | `true` or `false` |
| JSON | `json`, `jsonb` | `jsonb` is stored in a binary format and supports indexing |

{{% notice tip %}}
Avoid `char(n)` in PostgreSQL. It pads values with spaces and is usually slower than `text` or `varchar(n)`. See [Data Types](https://www.postgresql.org/docs/current/datatype.html) in the PostgreSQL documentation for the full list.
{{% /notice %}}
