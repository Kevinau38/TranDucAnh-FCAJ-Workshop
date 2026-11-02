---
title : "Connect to RDS PostgreSQL"
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

#### Overview

`rdspg-lab-db` runs in a private subnet without public access, so you cannot connect to it directly from the internet. In this chapter, you connect to the database in two ways:

| Method | Client | Path |
|---|---|---|
| pgAdmin 4 | Your computer | SSH tunnel through the bastion host |
| `psql` | App server in the private subnet | Directly inside the VPC |

Both methods only work because `rds-sg` allows port 5432 from `bastion-sg` and `app-sg`, which you created in chapter 2.
