---
title : "Preparation steps"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

#### Overview

In this chapter, you create all the AWS resources that the workshop uses, in the order of their dependencies:

| Resource | Purpose |
| --- | --- |
| VPC `rdspg-lab-vpc` | Network with public and private subnets in two Availability Zones |
| Security groups `bastion-sg`, `app-sg` | Control SSH access to the bastion host and the app server |
| Security group `rds-sg` | Allows PostgreSQL (port 5432) only from `bastion-sg` and `app-sg` |
| DB subnet group `rdspg-lab-subnet-group` | Tells RDS which private subnets to use |
| EC2 instances `rdspg-app-server`, `rdspg-bastion-host` | Clients that connect to the database |
| RDS DB instance `rdspg-lab-db` | The PostgreSQL database |

All resources are created in the **Asia Pacific (Singapore) ap-southeast-1** Region.
