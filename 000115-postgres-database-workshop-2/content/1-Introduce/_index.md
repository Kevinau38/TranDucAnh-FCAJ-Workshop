---
title : "Introduction"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

#### Overview

In the first workshop, you created an RDS for PostgreSQL database and connected to it. This workshop covers the **day-2 operations** of the same database: the tasks that keep a production database up to date, fast, protected, and available.

| Chapter | You learn to |
|---|---|
| 2. Database upgrade | Upgrade the minor version and the major version (17.9 → 17.11 → 18.6) |
| 3. Performance monitoring | Publish logs, create alarms, run a load test, find the bottleneck in Database Insights, scale up |
| 4. Backup and recovery | Use automated backups, snapshots, point-in-time recovery, and AWS Backup |
| 5. Scalability | Create and promote read replicas, migrate to a Multi-AZ DB cluster |
| 6. Parameter groups | Change the PostgreSQL configuration with a custom parameter group |
| 7. High availability | Convert to Multi-AZ and test a failover |
| 8. Learn more about PostgreSQL | Useful `pgbench` options and next steps |
| 9. Clean up | Delete every resource of the workshop |

#### Architecture

![Architecture](/images/1/architecture.png)

- **`rdspg-bastion-host`** in a public subnet is the only server that you reach from the internet, with SSH from your IP address only.
- **`rdspg-app-server`** in a private subnet runs `psql` and `pgbench`. You reach it through the bastion host with SSH `ProxyJump`.
- **`rdspg-lab-db`** is the RDS for PostgreSQL DB instance, in the private subnets. Its security group `rds-sg` allows port 5432 only from the App server and the bastion host.
- The App server installs packages from the Amazon Linux repositories through an **S3 gateway endpoint**, without a NAT gateway.
- The DB subnet group has private subnets in **three** Availability Zones, which a Multi-AZ DB cluster needs in chapter 5.

{{% notice note %}}
The workshop uses the Region Asia Pacific (Singapore) `ap-southeast-1` in the text and the diagram. The screenshots hide the Region, the account ID, and the endpoints. You can use any Region where RDS for PostgreSQL 17 and 18 are available.
{{% /notice %}}

#### Cost

Most resources of this workshop are in the AWS Free Tier: `db.t4g.micro`, `t3.micro`, 20 GiB of `gp3` storage, CloudWatch metrics, and Database Insights Standard. Some steps use resources that are **not** free and are charged per hour, for a few minutes only:

| Resource | Chapter |
|---|---|
| `db.t4g.xlarge` | 3.4 |
| Restored DB instances, read replica | 4.4, 4.5, 5.1 |
| Multi-AZ DB cluster (3 × `db.c6gd.medium`) | 5.4, 5.5 |
| Multi-AZ DB instance (standby) | 7 |

Delete each of them as soon as the step asks you to, and run chapter 9 at the end.

#### Content

1. [Preparation](1-1-preparation/)
