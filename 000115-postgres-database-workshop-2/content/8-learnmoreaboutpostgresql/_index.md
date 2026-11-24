---
title : "Learn more about PostgreSQL"
weight : 8
chapter : false
pre : " <b> 8. </b> "
---

#### Benchmark PostgreSQL with pgbench

`pgbench` ships with PostgreSQL and is the quickest way to put a repeatable load on a database. It opens a number of client sessions, runs the same transaction script in each session as fast as it can, and reports the throughput (**tps**, transactions per second) and the latency. `pgbench` does not measure CPU, memory, or disk I/O of the server: watch those in CloudWatch and Performance Insights while the test runs, as you did in chapter 3.

#### Built-in scripts

| Script | Option | What each transaction does |
|---|---|---|
| `tpcb-like` (default) | `-b tpcb-like` | 3 `UPDATE`, 1 `SELECT`, and 1 `INSERT`, loosely based on TPC-B |
| `simple-update` | `-N` or `-b simple-update` | Like `tpcb-like`, without the updates of `pgbench_tellers` and `pgbench_branches`, so less lock contention |
| `select-only` | `-S` or `-b select-only` | One `SELECT` by primary key. Useful to test reads, for example on a read replica |
| Your own script | `-f script.sql` | Any SQL that you write, with variables such as `\set aid random(1, 100000 * :scale)` |

#### Initialize the test data

The command that you ran in chapter 3:

```bash
pgbench -i --fillfactor=90 --scale=50 pglab
```

| Option | Meaning |
|---|---|
| `-i` (`--initialize`) | Creates the tables `pgbench_accounts`, `pgbench_branches`, `pgbench_tellers`, and `pgbench_history`. **It drops existing tables with these names first.** |
| `-F` (`--fillfactor=90`) | Creates the tables with a fill factor of 90: each page is filled to 90%, and the free space lets `UPDATE` keep new row versions on the same page (HOT updates) |
| `-s` (`--scale=50`) | Multiplies the data set: 50 branches, 500 tellers, and 5,000,000 accounts (about 750 MB) |
| `pglab` | The database to use. It must already exist |

#### Run the test

```bash
pgbench --protocol=prepared -P 30 --time=300 --client=50 --jobs=2 pglab
```

| Option | Meaning |
|---|---|
| `-M` (`--protocol=prepared`) | Sends the statements as prepared statements, so the server parses and plans them only once per session |
| `-P 30` (`--progress=30`) | Prints a progress line (tps, latency, standard deviation) every 30 seconds |
| `-T` (`--time=300`) | Runs the test for 300 seconds. Use `-t` to run a fixed number of transactions per client instead |
| `-c` (`--client=50`) | Opens 50 client sessions to the database |
| `-j` (`--jobs=2`) | Uses 2 threads in `pgbench`. Set it to the number of vCPUs of the client machine, not to the number of clients |
| `pglab` | The database to connect to |

The host, user, and password come from `PGHOST`, `PGUSER`, and `~/.pgpass`. Without them, add `-h <endpoint> -U postgres`. Note that `-p` is the **port**, not the protocol.

#### Tips

- **Change one thing at a time.** Run each test at least twice, and compare only tests where one setting changed: the instance class, a parameter, or the number of clients.
- **Increase the clients step by step** (for example 10, 25, 50, 100). When tps stops growing and latency rises, you have found the limit of the DB instance. `db.t4g.micro` allows only about 80 connections.
- **Watch burst credits.** `t` classes run on CPU credits. A long test can use them up, and the results then drop.
- **Do not test production.** `pgbench -i` drops tables, and the load can slow down real users.

#### Where to go next

- [Amazon RDS for PostgreSQL User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_PostgreSQL.html)
- [pgbench documentation](https://www.postgresql.org/docs/current/pgbench.html)
- [Best practices for Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_BestPractices.html)
- [Amazon RDS Proxy](https://aws.amazon.com/rds/proxy/) for connection pooling and faster failover
