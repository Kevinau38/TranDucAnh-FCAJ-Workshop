---
title : "Create a DB instance read replica of a Multi-AZ DB cluster"
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

#### Overview

The two readable standbys of a Multi-AZ DB cluster are in the same Region and use the reader endpoint. When you need more read capacity, or a copy of the data that you can scale on its own, you can add a **DB instance read replica** to the cluster.

In this step, you create the read replica `rdspg-lab-cluster-read` of `rdspg-lab-cluster`, check the replication, then delete the replica and the cluster to stop the cost.

{{% notice note %}}
To be a replication source, the Multi-AZ DB cluster must be **Available** and have automated backups turned on. Right after the promotion in step 5.4, RDS takes a first backup (**Backing-up**) and the **Create read replica** action is unavailable until it finishes.
{{% /notice %}}

#### Create the read replica

1. In the list of databases:
    - (1) Select `rdspg-lab-cluster`.
    - (2) Choose **Actions**.
    - (3) Choose **Create read replica**.

    ![Create read replica](/images/5/5-5/1.png)

2. On the **Create read replica** page:
    - (1) **Replica source** is `rdspg-lab-cluster`, with the role **Multi-AZ DB cluster**.
    - (2) **DB identifier**: `rdspg-lab-cluster-read`
    - (3) **DB instance class**: **Burstable classes**
    - (4) **Instance type**: `db.t4g.micro`. A read replica of a Multi-AZ DB cluster does not need to use the class of the cluster.

    ![Replica settings](/images/5/5-5/2.png)

3. Scroll down and check the other settings:
    - **Availability**: **Single-AZ DB instance deployment (1 instance)**
    - **DB subnet group**: `rdspg-lab-subnet-group`
    - **Public access**: **Not publicly accessible**
    - **Existing VPC security groups**: `rds-sg`

    Choose **Create read replica**.

4. In the list of databases:
    - (1) `rdspg-lab-cluster-read` appears under `rdspg-lab-cluster` with the role **Replica** and the status **Creating**.

    ![Creating the replica](/images/5/5-5/3.png)

5. After about 15 minutes, choose `rdspg-lab-cluster-read` and open the **Replication** tab:
    - (1) The breadcrumb shows that the replica belongs to `rdspg-lab-cluster`.
    - (2) The **Replication state** is **Replicating**.

    ![Replication](/images/5/5-5/4.png)

#### Delete the Multi-AZ DB cluster

The cluster and its replica are not in the free tier. Delete them now. You do not need a final snapshot.

6. Open [AWS CloudShell](https://console.aws.amazon.com/cloudshell/) and run:

    ```bash
    aws rds delete-db-instance --db-instance-identifier rdspg-lab-cluster-read \
        --skip-final-snapshot --delete-automated-backups \
        --query 'DBInstance.[DBInstanceIdentifier,DBInstanceStatus]' --output text

    aws rds delete-db-cluster --db-cluster-identifier rdspg-lab-cluster \
        --skip-final-snapshot --delete-automated-backups \
        --query 'DBCluster.[DBClusterIdentifier,Status]' --output text

    aws rds describe-db-clusters --db-cluster-identifier rdspg-lab-cluster \
        --query 'DBClusters[0].Status' --output text
    ```

    - (1) The replica `rdspg-lab-cluster-read` is **deleting**.
    - (2) RDS accepts the deletion of the cluster. The command still shows the status before the request, **available**.
    - (3) A few seconds later, the status of the cluster is **deleting**. RDS deletes its three DB instances with it. After about 10 minutes, `describe-db-clusters` returns *DBClusterNotFoundFault*.

    ![Delete the cluster](/images/5/5-5/5.png)

{{% notice tip %}}
In the console, you delete a Multi-AZ DB cluster by selecting the cluster (not its DB instances), choosing **Actions** > **Delete**, clearing **Create final snapshot**, and typing `delete me`.
{{% /notice %}}

#### (Optional) AWS CLI

{{%expand "Create the read replica with the AWS CLI" %}}
```bash
aws rds create-db-instance-read-replica \
    --db-instance-identifier rdspg-lab-cluster-read \
    --source-db-cluster-identifier rdspg-lab-cluster \
    --db-instance-class db.t4g.micro \
    --db-subnet-group-name rdspg-lab-subnet-group \
    --vpc-security-group-ids <ID of rds-sg> \
    --no-publicly-accessible
```
{{% /expand %}}
