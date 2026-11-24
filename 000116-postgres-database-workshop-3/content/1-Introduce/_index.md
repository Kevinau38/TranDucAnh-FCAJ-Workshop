---
title : "Introduction"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

#### Overview

The first two workshops looked at RDS for PostgreSQL from the side of the database administrator. This workshop looks at it from the side of the **application developer**: how your code finds the database, how it authenticates, and how it keeps the connection secure.

| Chapter | You learn to |
|---|---|
| 2. Preparation | Create the network, the database with an RDS-managed secret, and a development server with an IAM role |
| 3. Code examples | Connect from Node.js with environment variables, then with the AWS SDK and Secrets Manager |
| 4. Deploy the application | Run the FCJ User Management web application on the development server |
| 5. Clean up | Delete every resource of the workshop |

#### Architecture

![Architecture](/images/1/architecture.png)

- **`rdspg-dev-server`** is an EC2 instance in a public subnet. You open a shell on it with **AWS Systems Manager Session Manager**, so it has no SSH key and no SSH port open. Its security group `dev-server-sg` allows only port 3000 (the web application) from your IP address.
- The instance profile gives the server the IAM role **`FCJ-rdspg-dev-ec2-role`**. The application uses its temporary credentials to call the RDS API and Secrets Manager; there are no access keys on the server.
- **`rdspg-dev-db`** is an RDS for PostgreSQL 18 DB instance in two private subnets. Its security group `rds-dev-sg` allows port 5432 only from `dev-server-sg`.
- RDS stores the master user and its password in a **secret in AWS Secrets Manager** (`rds!db-...`), and rotates the password every 7 days.

| Tool | Version in this workshop |
|---|---|
| Amazon Linux | 2023 |
| Node.js / npm | 24 / 11 (`nodejs24` package) |
| PostgreSQL client / server | 18.6 / 18.3 |
| `pg`, `express`, `express-handlebars` | 8.x, 4.x, 8.x |
| AWS SDK for JavaScript | v3 (`@aws-sdk/client-rds`, `@aws-sdk/client-secrets-manager`) |

{{% notice note %}}
The workshop uses the Region Asia Pacific (Singapore) `ap-southeast-1` in the text and the scripts. The screenshots hide the Region, the account ID, the public IP addresses, and the endpoints. You can use any Region where RDS for PostgreSQL 18 is available.
{{% /notice %}}

#### Cost

`db.t4g.micro`, `t3.micro`, and 20 GiB of `gp3` storage are in the AWS Free Tier. You pay for the RDS-managed secret (USD 0.40 per month, charged per hour) and for the public IPv4 address of the server (USD 0.005 per hour). Run chapter 5 when you finish.

Continue with [2. Preparation](../2-preparation/).
