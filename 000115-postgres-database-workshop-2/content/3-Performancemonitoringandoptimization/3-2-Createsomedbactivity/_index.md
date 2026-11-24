---
title : "Generate database activity"
weight : 2
chapter : false
pre : " <b> 3.2. </b> "
---

#### Overview

`pgbench` runs the same small transaction many times in parallel and reports how many transactions per second (tps) the database completes. The default transaction, similar to the TPC-B benchmark, updates three tables and inserts one row into a fourth:

| Table | Rows at scale 1 | Rows at scale 50 |
|---|---|---|
| `pgbench_branches` | 1 | 50 |
| `pgbench_tellers` | 10 | 500 |
| `pgbench_accounts` | 100,000 | 5,000,000 |
| `pgbench_history` | 0 | Grows with each transaction |

You run all commands on the App server. `PGHOST`, `PGUSER`, and `~/.pgpass` from the Preparation step let `pgbench` connect without asking for the password.

#### Create the test data

1. Connect to the App server (`ssh rdspg-app`), then run:

    ```bash
    time pgbench -i --fillfactor=90 --scale=50 pglab
    ```

    - (1) `pgbench` drops old test tables, creates the four tables, generates the data, runs `VACUUM`, and creates the primary keys.
    - (2) With scale 50, the test data is about 750 MB and takes less than a minute to create.

    ![pgbench initialization](/images/3/3-2/1.png)

#### Run a load test

2. Run a 5-minute load test with 50 concurrent clients:

    ```bash
    pgbench --protocol=prepared -P 30 --time=300 --client=50 --jobs=2 pglab
    ```

    - (1) Every 30 seconds, `pgbench` prints the throughput and the latency of the last interval.
    - (2) At the end, `pgbench` prints the summary. On `db.t4g.micro`, the result is about **337 tps** with an average latency of about **148 ms**.

    ![Load test on db.t4g.micro](/images/3/3-2/2.png)

{{% notice note %}}
Your numbers can be different, because `db.t4g.micro` is a burstable instance: its performance depends on the CPU credits that the instance has collected. Keep your result, you compare it with the second load test in step 3.4.
{{% /notice %}}

While the load test runs, continue with the next step to watch the database in Database Insights.

{{% notice warning %}}
`db.t4g.micro` allows about 80 connections. Do not set `--client` higher than about 70, or `pgbench` fails with *too many clients already*.
{{% /notice %}}
