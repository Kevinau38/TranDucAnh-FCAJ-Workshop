---
title : "AWS RDS PostgreSQL"
weight : 1
chapter : false
---
# Operating Amazon RDS for PostgreSQL

#### Overview

In this workshop, you operate an Amazon RDS for PostgreSQL DB instance through its whole life: you upgrade the engine, monitor and tune performance, back up and restore data, scale reads and writes, change the configuration with parameter groups, and test high availability with a real failover.

![Workshop architecture](/images/1/architecture.png)

#### What you learn

- Upgrade PostgreSQL from version 17 to 18 with little risk, and check the result.
- Find slow queries with CloudWatch, Performance Insights, and `pgbench`.
- Protect data with automated backups, snapshots, point-in-time recovery, and AWS Backup.
- Scale with read replicas, larger instance classes, and Multi-AZ DB clusters.
- Change engine settings with custom parameter groups.
- Make the database highly available with Multi-AZ, and watch a failover from the application side.
