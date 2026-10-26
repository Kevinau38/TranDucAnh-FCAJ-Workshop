---
title : "Create RDS Security Group"
weight : 3
chapter : false
pre : " <b> 2.3. </b> "
---

#### Overview

In this step, you create `rds-sg` for the Amazon RDS for PostgreSQL instance. It allows PostgreSQL connections (port **5432**) only from two sources:

| Source | Reason |
|---|---|
| `app-sg` | The App server connects to the database with `psql` |
| `bastion-sg` | pgAdmin on your computer connects through an SSH tunnel, so the connection reaches RDS from the bastion host |

{{% notice info %}}
`rds-sg` does not allow any IP address, including your own. Your computer reaches the database only through the bastion host.
{{% /notice %}}

#### Create rds-sg

1. In the [Amazon EC2 console](https://console.aws.amazon.com/ec2/), choose **Security Groups**, then choose **Create security group**. In **Basic details**:
    - (1) **Security group name**: `rds-sg`
    - (2) **Description**: `Allow PostgreSQL from app server and bastion host`
    - (3) **VPC**: `rdspg-lab-vpc`

    ![rds-sg basic details](/images/2-preparation/2.3/1.png)

2. In **Inbound rules**, add two rules:
    - (1) **Type**: `PostgreSQL`, **Source type**: `Custom`, **Source**: `app-sg`
    - (2) **Type**: `PostgreSQL`, **Source type**: `Custom`, **Source**: `bastion-sg`

    ![rds-sg inbound rules](/images/2-preparation/2.3/2.png)

{{% notice note %}}
When you choose the **PostgreSQL** type, the protocol is set to TCP and the port to 5432 automatically.
{{% /notice %}}

3. Keep the default outbound rules and choose **Create security group**. `rds-sg` now has two inbound rules on port 5432.

    ![rds-sg created](/images/2-preparation/2.3/3.png)
