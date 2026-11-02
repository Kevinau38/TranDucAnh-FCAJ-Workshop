---
title : "Introduction"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

#### PostgreSQL

[PostgreSQL](https://aws.amazon.com/rds/postgresql/what-is-postgresql/) is an open source object-relational database system with more than 35 years of active development. It is known for its reliability, standards compliance (SQL), and extensibility: you can add data types, functions, and extensions such as PostGIS or pgvector. Many enterprises and startups use PostgreSQL as the main database for business, web, and mobile applications.

#### Amazon RDS for PostgreSQL

[Amazon RDS for PostgreSQL](https://aws.amazon.com/rds/postgresql/) is a managed database service. AWS runs the PostgreSQL engine for you, so you can focus on your data and your application instead of the database servers.

| AWS manages | You manage |
| --- | --- |
| Hardware, operating system, and PostgreSQL installation | Database design: databases, schemas, tables, and indexes |
| Minor version patches during the maintenance window | Users, roles, and privileges inside PostgreSQL |
| Automated backups and point-in-time recovery | Network access: VPC, subnets, and security groups |
| Multi-AZ failover and read replicas, when you enable them | Choosing the instance class, storage, and major version |
| Monitoring with Amazon CloudWatch and CloudWatch Database Insights | Query performance and application connections |

Amazon RDS supports the community PostgreSQL major versions 14, 15, 16, 17, and 18. The code, applications, and tools that you already use with PostgreSQL work with Amazon RDS without changes. Because the service is managed, the master user receives the `rds_superuser` role instead of the PostgreSQL `superuser`. You see what this means in chapter 4.

{{% notice info %}}
Check the [Release calendars for Amazon RDS for PostgreSQL](https://docs.aws.amazon.com/AmazonRDS/latest/PostgreSQLReleaseNotes/postgresql-release-calendar.html) for the current list of versions and their end of standard support dates.
{{% /notice %}}

#### Workshop architecture

![Workshop architecture](/images/1-introduction/architecture.png)

- The VPC `rdspg-lab-vpc` has a public subnet and a private subnet in each of two Availability Zones.
- The RDS DB instance `rdspg-lab-db` runs in a private subnet and has no public access. RDS requires a DB subnet group with subnets in at least two Availability Zones.
- The bastion host in the public subnet is the only entry point from the internet. It accepts SSH only from your IP address.
- The app server runs in a private subnet without internet access. It installs packages through the S3 gateway endpoint.
- Security groups use other security groups as sources: `rds-sg` accepts port 5432 only from `bastion-sg` and `app-sg`.

#### What you do in this workshop

| Chapter | Content |
| --- | --- |
| 2. Preparation steps | Create the VPC, security groups, DB subnet group, EC2 instances, and the RDS DB instance |
| 3. Connect to RDS PostgreSQL | Connect with pgAdmin 4 through an SSH tunnel, and with `psql` from the app server |
| 4. Start with PostgreSQL | Work with databases, schemas, tables, roles, the system catalog, and queries |
| 5. Helpful resources | Documentation and blog posts to continue learning |
| 6. Clean up resources | Delete all resources to avoid charges |

#### Prerequisites

- An AWS account with permissions for Amazon VPC, Amazon EC2, and Amazon RDS.
- A Windows computer with PowerShell. OpenSSH Client is included in Windows 10 and Windows 11.
- Basic knowledge of SQL.

{{% notice warning %}}
The resources in this workshop are billed while they run, including the public IPv4 address of the bastion host. Complete chapter 6 when you finish.
{{% /notice %}}
