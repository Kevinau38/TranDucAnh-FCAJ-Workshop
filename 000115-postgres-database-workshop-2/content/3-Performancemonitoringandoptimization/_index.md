---
title : "Performance monitoring and optimization"
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

#### Overview

Monitoring the health and performance of a database is an everyday task. In this chapter, you:

- Publish the PostgreSQL log to Amazon CloudWatch Logs and create a CloudWatch alarm that sends you an email when CPU utilization is high.
- Generate test data and run a load test with `pgbench`, the benchmarking tool that ships with PostgreSQL.
- Find the bottleneck with **CloudWatch Database Insights**.
- Scale the DB instance up, run the same load test again, and compare the results.

| Tool | What it shows | Cost in this workshop |
|---|---|---|
| CloudWatch metrics | CPU, memory, storage, I/O, connections, every minute | Free |
| CloudWatch Logs | The PostgreSQL log and the upgrade log | A few cents for the log data |
| CloudWatch alarm + Amazon SNS | Email notification when a metric crosses a threshold | Free within the free tier |
| Database Insights - Standard | Database load by wait event, top SQL statements, 7 days of history | Free |

{{% notice info %}}
**Performance Insights** has been replaced by **CloudWatch Database Insights**. The **Standard** mode that you enabled when you created the database keeps 7 days of history at no extra cost. The **Advanced** mode adds fleet-wide views and longer retention, and is charged per vCPU.
{{% /notice %}}
