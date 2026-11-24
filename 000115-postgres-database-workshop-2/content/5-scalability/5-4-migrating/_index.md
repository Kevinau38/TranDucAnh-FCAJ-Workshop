---
title : "Migrate to a Multi-AZ DB cluster with a read replica"
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

#### Overview

A **Multi-AZ DB cluster** has one writer DB instance and **two readable standby** DB instances, each in a different Availability Zone. Compared with a Multi-AZ DB instance (one primary and one standby that you cannot read), it gives you:

| | Multi-AZ DB instance | Multi-AZ DB cluster |
|---|---|---|
| DB instances | 1 primary + 1 standby | 1 writer + 2 readable standbys |
| Read from the standby | No | Yes, through the **reader endpoint** |
| Typical failover time | 60 to 120 seconds | Under 35 seconds |
| Storage | EBS | Local NVMe SSD + EBS |
| Instance classes | All | `m5d`, `m6gd`, `m6id`, `r5d`, `r6gd`, `r6id`, `c6gd`... |

To move an existing Single-AZ or Multi-AZ DB instance to a Multi-AZ DB cluster with little downtime, you create a **Multi-AZ DB cluster read replica** of the DB instance, wait until it catches up, stop the writes, and **promote** the cluster.

{{% notice warning %}}
A Multi-AZ DB cluster needs subnets in **three** Availability Zones in its DB subnet group. If `rdspg-lab-subnet-group` has only two subnets, add a third private subnet first (see the note below). The cluster runs three DB instances that are not in the free tier: delete it as soon as you finish step 5.5.
{{% /notice %}}

{{% notice note %}}
If the console shows *Can't create a Multi AZ DB cluster because there aren't enough subnets available*, add a private subnet in a third Availability Zone in AWS CloudShell, then choose **Create read replica** again:

```bash
VPC=<ID of rdspg-lab-vpc>
RT=<ID of the private route table>
PRI3=$(aws ec2 create-subnet --vpc-id $VPC --cidr-block 10.0.160.0/20 \
    --availability-zone <third AZ> \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=rdspg-lab-subnet-private3}]' \
    --query Subnet.SubnetId --output text)
aws ec2 associate-route-table --route-table-id $RT --subnet-id $PRI3
aws rds modify-db-subnet-group --db-subnet-group-name rdspg-lab-subnet-group \
    --subnet-ids <private subnet 1> <private subnet 2> $PRI3
```
{{% /notice %}}

#### Create the Multi-AZ DB cluster read replica

1. In the list of databases, select `rdspg-lab-db`, then choose **Actions** > **Create read replica**, as in step 5.1.

2. Scroll down to **Availability** first, because this choice changes the rest of the page:
    - (1) Choose **Multi-AZ DB cluster deployment (3 instances)**.

    ![Multi-AZ DB cluster](/images/5/5-4/1.png)

3. Scroll back up:
    - (1) **DB cluster identifier**: `rdspg-lab-cluster`
    - (2) **DB instance class**: **Compute optimized classes**
    - (3) **Instance type**: `db.c6gd.medium`, the smallest class for a Multi-AZ DB cluster

    ![Cluster settings](/images/5/5-4/2.png)

4. In **Connectivity**, check that the **DB subnet group** is `rdspg-lab-subnet-group`, **Public access** is **Not publicly accessible**, and the security group is `rds-sg`. Choose **Create read replica**.

5. In the list of databases:
    - (1) `rdspg-lab-cluster` appears under `rdspg-lab-db` with the role **Multi-AZ DB cluster** and the size **3 instances**.
    - (2) RDS creates three **Reader instances** `db.c6gd.medium`, in three Availability Zones.

    ![Creating the cluster](/images/5/5-4/3.png)

    The cluster takes about 20 to 30 minutes.

6. When the **Status** of `rdspg-lab-cluster` is **Available**, choose it and open the **Connectivity & security** tab:
    - (1) The **Writer** endpoint (`rdspg-lab-cluster.cluster-...`) always points to the writer DB instance. Use it for reads and writes.
    - (2) The **Reader** endpoint (`rdspg-lab-cluster.cluster-ro-...`) spreads connections across the readable standbys.

    ![Cluster endpoints](/images/5/5-4/4.png)

7. Open the **Replication** tab:
    - (1) The cluster has the role **Multi-AZ DB replica cluster**, its DB instances are in three Availability Zones, and its source is `rdspg-lab-db`, with the state **Replicating**.

    ![Cluster replication](/images/5/5-4/5.png)

#### Promote the Multi-AZ DB cluster

In a real migration, you now stop the writes on `rdspg-lab-db`, wait until **Replica lag** is 0, then promote the cluster and point the application to the writer endpoint.

8. In the list of databases:
    - (1) Select `rdspg-lab-cluster`.
    - (2) Choose **Actions**.
    - (3) Choose **Promote**.

    ![Promote the cluster](/images/5/5-4/6.png)

9. On the **Promote read replica: rdspg-lab-cluster** page:
    - (1) Read the message: replication stops as soon as the promotion begins.
    - (2) Choose **Promote read replica**.

    ![Promote read replica](/images/5/5-4/7.png)

10. The **Status** of `rdspg-lab-cluster` changes to **Promoting**.

    ![Promoting](/images/5/5-4/8.png)

11. After a few minutes, the promotion is complete:
    - (1) `rdspg-lab-cluster` is no longer under `rdspg-lab-db`. It is now a standalone **Multi-AZ DB cluster** that accepts writes. RDS takes a first backup (**Backing-up**), then the **Status** is **Available**.
    - (2) The role of `rdspg-lab-db` changes from **Primary** to **Instance**, because it no longer has a replica.

    ![Promoted cluster](/images/5/5-4/9.png)

{{% notice tip %}}
You can also convert a DB instance to a Multi-AZ DB cluster from a snapshot (**Restore snapshot** > **Multi-AZ DB cluster**). The read replica method keeps the downtime to the few minutes of the promotion.
{{% /notice %}}

#### (Optional) AWS CLI

{{%expand "Create and promote a Multi-AZ DB cluster read replica with the AWS CLI" %}}
```bash
aws rds create-db-cluster \
    --db-cluster-identifier rdspg-lab-cluster \
    --replication-source-identifier <ARN of rdspg-lab-db> \
    --engine postgres \
    --db-cluster-instance-class db.c6gd.medium \
    --storage-type gp3 \
    --db-subnet-group-name rdspg-lab-subnet-group \
    --vpc-security-group-ids <ID of rds-sg> \
    --no-publicly-accessible

aws rds wait db-cluster-available --db-cluster-identifier rdspg-lab-cluster

aws rds promote-read-replica-db-cluster --db-cluster-identifier rdspg-lab-cluster
```
{{% /expand %}}
