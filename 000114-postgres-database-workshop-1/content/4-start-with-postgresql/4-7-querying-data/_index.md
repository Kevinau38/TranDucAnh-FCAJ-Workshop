---
title : "Querying Data"
weight : 7
chapter : false
pre : " <b> 4.7. </b> "
---

#### Overview

In this step, you practice everything from this chapter on a sample table `fcj_users`: create a table, insert data, and query it with filters, sorting, and limits.

Run each statement in the Query Tool of **first_database**: (1) enter the statement, (2) choose **Execute**.

#### Create the table and insert data

1. Run the following block. `RESET search_path` returns `search_path` to the default value after step 4.6, so the table is created in the `public` schema.

    ```sql
    RESET search_path;

    CREATE TABLE fcj_users (
        user_id       integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
        first_name    varchar(100),
        last_name     varchar(100),
        gender        char(1),
        date_of_birth date,
        nationality   varchar(20)
    );

    INSERT INTO fcj_users (first_name, last_name, gender, date_of_birth, nationality) VALUES
    ('Malin','Akerman','F','1978-05-12','Swedish'),
    ('Tim','Allen','M','1953-06-13','American'),
    ('Julie','Andrews','F','1935-10-01','American'),
    ('Ivana','Baquero','F','1994-06-11','British'),
    ('Lorraine','Bracco','F','1954-10-02','Mexican'),
    ('Alice','Braga','F','1983-04-15','Japanese'),
    ('Marlon','Brando','M','1924-04-03','American'),
    ('Adrien','Brody','M','1973-04-14','Mexican'),
    ('Peter','Carlberg','M','1950-12-08','Chinese'),
    ('Gemma','Chan','F','1982-11-29','Japanese'),
    ('Chen','Chang','M','1976-10-14','Mexican'),
    ('Graham','Chapman','M','1941-01-08','Japanese'),
    ('Pei-pei','Cheng','F','1946-12-04','Mexican'),
    ('Maggie','Cheung','F','1964-09-20','American'),
    ('Min-sik','Choi','M','1962-05-30','Chinese'),
    ('Yun-fat','Chow','M','1955-05-18','American'),
    ('John','Cleese','M','1939-10-27','British'),
    ('Paddy','Considine','M','1973-09-05','Mexican'),
    ('Abbie','Cornish','F','1982-08-07','Swedish'),
    ('Brian','Cox','M','1946-06-01','Swedish'),
    ('Scatman','Crothers','M','1910-05-23','American'),
    ('Russell','Crowe','M','1964-04-07','American'),
    ('Tom','Cruise','M','1962-07-03','Swedish'),
    ('Darlan','Cunha','M','1988-07-26','American'),
    ('Willem','Dafoe','M','1955-07-22','British'),
    ('Paul','Dano','M','1984-06-19','Swedish'),
    ('Daniel','Day-Lewis','M','1957-04-29','Swedish'),
    ('Robert','De Niro','M','1943-08-17','British'),
    (NULL,'Denden','M','1950-01-23','Mexican'),
    ('Leonardo','DiCaprio','M','1974-11-11','American'),
    ('Peter','Dinklage','M','1969-06-11','British'),
    ('Hiroki','Doi','M','1999-08-10','Japanese'),
    ('Kirsten','Dunst','F','1982-04-30','Swedish'),
    ('Shelley','Duvall','F','1949-07-07','British'),
    ('Ralph','Fiennes','M','1962-12-22','British'),
    ('Leandro','Firmino','M','1978-06-23','Swedish'),
    ('Carrie','Fisher','F','1956-10-21','American'),
    ('Harrison','Ford','M','1942-07-13','American'),
    ('Jodie','Foster','F','1962-11-19','Chinese');
    ```

    The **Messages** tab shows `INSERT 0 39`.

    ![Insert data](/images/4-postgresql/4.7/1.png)

#### Query the data

2. Show each user's full name and nationality. The `||` operator joins strings.

    ```sql
    SELECT first_name || ' ' || last_name AS full_name, nationality FROM fcj_users;
    ```

    ![Full name](/images/4-postgresql/4.7/2.png)

3. Find users who are Swedish or American and were born from 1980 to 1990, newest first:

    ```sql
    SELECT * FROM fcj_users
    WHERE nationality IN ('Swedish', 'American')
      AND date_of_birth BETWEEN '1980-01-01' AND '1990-12-31'
    ORDER BY date_of_birth DESC;
    ```

    ![Filter and sort](/images/4-postgresql/4.7/3.png)

{{% notice warning %}}
`AND` has a higher priority than `OR`. Writing `nationality = 'Swedish' OR nationality = 'American' AND date_of_birth BETWEEN ...` returns every Swedish user regardless of the date. Use `IN (...)` or parentheses to group the `OR` conditions.
{{% /notice %}}

4. List the 5 youngest users:

    ```sql
    SELECT * FROM fcj_users ORDER BY date_of_birth DESC FETCH FIRST 5 ROWS ONLY;
    ```

    ![Youngest users](/images/4-postgresql/4.7/4.png)

{{% notice tip %}}
To list the 5 oldest users, change `DESC` to `ASC`. `FETCH FIRST n ROWS ONLY` is the SQL-standard form of `LIMIT n`.
{{% /notice %}}

5. Find rows with missing values:

    ```sql
    SELECT * FROM fcj_users
    WHERE date_of_birth IS NULL OR first_name IS NULL OR last_name IS NULL;
    ```

    Use `IS NULL` instead of `= NULL`, because any comparison with `NULL` returns unknown, not true.

    ![Missing values](/images/4-postgresql/4.7/5.png)
