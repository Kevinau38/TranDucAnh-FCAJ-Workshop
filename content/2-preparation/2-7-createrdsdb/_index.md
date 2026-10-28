---
title : "Create an RDS PostgreSQL DB"
weight : 7
chapter : false
pre : " <b> 2.7. </b> "
---

#### Overview

In this step, you create `rdspg-lab-db`, an Amazon RDS for PostgreSQL DB instance in the private subnets, using the network and security groups from the previous steps.

| Setting | Value | Reason |
|---|---|---|
| Creation method | Full configuration | You choose the VPC, DB subnet group, and security group yourself |
| Engine version | PostgreSQL 18.6-R1 | The latest version available when this workshop was updated |
| Template | Sandbox | Limits the options to low-cost settings for learning |
| Deployment | Single-AZ DB instance | A standby instance is not needed for this workshop |
| Instance class | `db.t4g.micro` | Free plan eligible, Graviton-based |
| Storage | gp3, 20 GiB, autoscaling off | The minimum size, enough for this workshop |
| Credentials | Self managed | Secrets Manager charges for each secret per month |
| Compute resource | Don't connect to an EC2 compute resource | You already created `rds-sg` in step 2.3 |
| Public access | No | The database is reachable only from inside the VPC |
| Security group | `rds-sg` only | Allows port 5432 only from `app-sg` and `bastion-sg` |
| Deletion protection | Off | So that you can delete the database in the clean-up step |

#### Choose the engine and template

1. Open the [Amazon RDS console](https://console.aws.amazon.com/rds/):
    - (1) Choose **Databases**.
    - (2) Choose **Create database**.
    - (3) Choose **Full configuration**.

    ![Create database](/images/2-preparation/2.7/1.png)

2. Choose the engine and template:
    - (1) **Engine type**: **PostgreSQL**
    - (2) **Database creation method**: **Full configuration**
    - (3) **Templates**: **Sandbox**

    ![Engine and template](/images/2-preparation/2.7/2.png)

{{% notice warning %}}
The default template is **Production**, which selects Multi-AZ and larger resources. Make sure you choose **Sandbox**.
{{% /notice %}}

3. In **Availability and durability**:
    - (1) Choose **Single-AZ DB instance deployment (1 instance)**.

    ![Availability and durability](/images/2-preparation/2.7/3.png)

{{% notice info %}}
With the Sandbox template, the Multi-AZ options are disabled. A **Multi-AZ DB instance** has one primary and one standby that cannot serve reads. A **Multi-AZ DB cluster** has one primary and two readable standbys. A **Single-AZ** instance has no standby and no data redundancy, which is acceptable for a lab but not for production.
{{% /notice %}}

#### Configure the settings and credentials

4. In **Settings**:
    - (1) **Engine version**: **PostgreSQL 18.6-R1**
    - (2) Keep **Enable RDS Extended Support** cleared.
    - (3) **DB instance identifier**: `rdspg-lab-db`
    - (4) **Master username**: `postgres`

    ![Settings](/images/2-preparation/2.7/4.png)

{{% notice note %}}
**RDS Extended Support** is a paid offering that keeps a major version running after its end of standard support. PostgreSQL 18 is new, so you do not need it.
{{% /notice %}}

5. In **Credentials management**:
    - (1) Choose **Self managed**.
    - (2) Keep **Auto generate password** cleared.
    - (3) Enter a **Master password**.
    - (4) Enter the same password in **Confirm master password**.
    - (5) **Database authentication options**: **Password authentication**

    ![Credentials](/images/2-preparation/2.7/5.png)

{{% notice warning %}}
Save the master password in a safe place, outside the workshop folder. You need it to connect to the database, and RDS does not show it again. The password cannot contain `/`, `'`, `"`, or `@`.
{{% /notice %}}

#### Configure the instance and storage

6. In **Instance configuration** and **Storage**:
    - (1) Choose **Burstable classes (includes t classes)**.
    - (2) **Instance type**: `db.t4g.micro`
    - (3) **Storage type**: **General Purpose SSD (gp3)**
    - (4) **Allocated storage**: `20` GiB

    ![Instance and storage](/images/2-preparation/2.7/6.png)

7. In **Additional storage configuration**:
    - (1) Clear **Enable storage autoscaling**.

    ![Storage autoscaling](/images/2-preparation/2.7/7.png)

{{% notice tip %}}
With gp3 storage below 400 GiB, RDS includes a baseline of 3,000 IOPS and 125 MiBps at no extra cost. Storage autoscaling is turned off because RDS cannot reduce storage after it grows.
{{% /notice %}}

#### Configure connectivity

8. In **Connectivity**:
    - (1) **Compute resource**: **Don't connect to an EC2 compute resource**
    - (2) **Network type**: **IPv4**
    - (3) **Virtual private cloud (VPC)**: `rdspg-lab-vpc`
    - (4) **DB subnet group**: `rdspg-lab-subnet-group`

    ![Connectivity](/images/2-preparation/2.7/8.png)

{{% notice warning %}}
You cannot change the VPC after the database is created.
{{% /notice %}}

9. Continue in **Connectivity**:
    - (1) **Public access**: **No**
    - (2) **VPC security group (firewall)**: **Choose existing**
    - (3) **Existing VPC security groups**: choose only `rds-sg`. Remove `default` if it is selected.
    - (4) **Availability Zone**: **No preference**

    ![Public access and security group](/images/2-preparation/2.7/9.png)

10. In **Additional configuration** of **Connectivity**:
    - (1) Keep **Database port** as `5432`.

    ![Database port](/images/2-preparation/2.7/10.png)

{{% notice note %}}
Keep **Create an RDS Proxy** cleared and **Certificate authority** as the default. The inbound rules of `rds-sg` allow port 5432, so do not change the port.
{{% /notice %}}

#### Configure monitoring and create the database

11. In **Monitoring**:
    - (1) Keep **Database Insights - Standard**.
    - (2) Keep **Retention period** as **7 days (free)**.

    ![Monitoring](/images/2-preparation/2.7/11.png)

{{% notice info %}}
Keep the following default settings:
- **Enhanced Monitoring**, **Log exports**, and **DevOps Guru**: off, because they add charges.
- **Initial database name**: empty. PostgreSQL always includes the default `postgres` database.
- **DB parameter group**: `default.postgres18`. A parameter group holds the settings of the whole DB instance.
- **Encryption**: on, with the AWS managed key `aws/rds`.
- **Automated backup**: on, with a retention period of 1 day.
- **Auto minor version upgrade**: on. RDS applies minor versions (for example, 18.6 to 18.7) during the maintenance window.
{{% /notice %}}

12. At the end of the page:
    - (1) Keep **Enable deletion protection** cleared.
    - (2) Choose **Create database**.

    ![Create database](/images/2-preparation/2.7/12.png)

13. Wait about 10 to 15 minutes until the **Status** of `rdspg-lab-db` is **Available**.

    ![Database available](/images/2-preparation/2.7/13.png)

#### Verify the database

14. Choose `rdspg-lab-db`, then open the **Connectivity & security** tab:
    - (1) In **Connect using**, choose **Code snippets**.

    The **Connection steps** show the endpoint and the `psql` command that you use in a later step. **Internet access gateway** is **Disabled**.

    ![Code snippets](/images/2-preparation/2.7/14.png)

15. In **Connect using**:
    - (1) Choose **Endpoints**.

    ![Endpoints](/images/2-preparation/2.7/15.png)

16. In **Additional configurations**, check the following:
    - **Endpoint** and **Port** `5432`
    - **VPC**: `rdspg-lab-vpc` and **Subnet group**: `rdspg-lab-subnet-group`
    - **VPC security groups**: `rds-sg`, **Active**
    - **Publicly accessible**: **No**

    ![Connectivity and security](/images/2-preparation/2.7/16.png)

17. In **Security group rules**, check that `rds-sg` has two inbound rules, one from `app-sg` and one from `bastion-sg`.

    ![Security group rules](/images/2-preparation/2.7/17.png)

{{% notice tip %}}
Copy the **Endpoint** and keep it with your master password. Every connection in the next chapters uses it.
{{% /notice %}}

{{% notice warning %}}
From now on, the RDS instance and the two EC2 instances are charged per hour. If you pause the workshop for a long time, stop them. RDS starts a stopped DB instance again automatically after 7 days.
{{% /notice %}}
