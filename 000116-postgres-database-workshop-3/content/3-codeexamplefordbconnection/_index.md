---
title : "Code example for database connection"
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

In this chapter you write two small Node.js programs that connect to `rdspg-dev-db`. Both verify the TLS certificate of the server with the RDS certificate bundle. They differ in **where the connection settings come from**:

| | [3.1 Simple database connection](3-1-sampledbconnect/) | [3.2 Auto-fetching instance credentials and connecting](3-2-autofetchingcredential/) |
|---|---|---|
| Endpoint | Environment variable `PGHOST` | `DescribeDBInstances` (RDS API) |
| User name and password | Environment variables `PGUSER`, `PGPASSWORD` | `GetSecretValue` (Secrets Manager) |
| What the code needs | The password, copied by you | Only the DB instance identifier and the IAM role |
| Password rotation | Breaks the program until you copy the new password | Picked up automatically |

Run every command in the Session Manager terminal of `rdspg-dev-server`, as `ec2-user`, in the folder `~/rdspg-dev-app` that you created in [2.1](../2-preparation/2-1-setupenv/).
