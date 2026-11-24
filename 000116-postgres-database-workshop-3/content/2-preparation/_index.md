---
title : "Preparation"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

#### Overview

In this chapter, you create the environment of the workshop and look at how RDS keeps the database password in AWS Secrets Manager.

| Resource | Name | Settings |
|---|---|---|
| VPC | `rdspg-dev-vpc` | `10.0.0.0/16`, 1 public subnet, 2 private subnets, internet gateway |
| Security groups | `dev-server-sg`, `rds-dev-sg` | Port 3000 from your IP → dev server; PostgreSQL 5432 from the dev server |
| IAM role | `FCJ-rdspg-dev-ec2-role` | Session Manager, `rds:DescribeDBInstances` on `rdspg-dev-db`, `secretsmanager:GetSecretValue` on RDS-managed secrets |
| EC2 | `rdspg-dev-server` | Amazon Linux 2023, `t3.micro`, no key pair |
| RDS | `rdspg-dev-db` | PostgreSQL 18, `db.t4g.micro`, master password managed in Secrets Manager, database `pglab` |
