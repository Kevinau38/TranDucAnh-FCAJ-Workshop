---
title : "Clean up resources"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---

#### Overview

To avoid unexpected charges, delete every resource you created in this workshop. The Amazon RDS DB instance and the EC2 instances are billed while they run, and the public IPv4 address of the bastion host is billed per hour.

Delete the resources in this order. A resource cannot be deleted while another resource still uses it:

| Order | Resource | Reason |
| --- | --- | --- |
| 1 | RDS DB instance `rdspg-lab-db` | Uses the DB subnet group and `rds-sg` |
| 2 | EC2 instances `rdspg-bastion-host`, `rdspg-app-server` | Use `bastion-sg`, `app-sg`, and the key pair |
| 3 | DB subnet group `rdspg-lab-subnet-group` | Can be deleted only after the DB instance is deleted |
| 4 | Security groups `rds-sg`, `app-sg`, `bastion-sg` | Reference each other in their rules |
| 5 | Key pair `rdspg-lab-key` | |
| 6 | VPC `rdspg-lab-vpc` | Deleted together with its subnets, route tables, internet gateway, and S3 gateway endpoint |

#### Delete the RDS DB instance

1. Open the [Amazon RDS console](https://console.aws.amazon.com/rds/):
    - (1) Choose **Databases**.
    - (2) Select `rdspg-lab-db`.
    - (3) Choose **Actions**.
    - (4) Choose **Delete**.

    ![Delete DB instance](/images/6-cleanup/1.png)

2. In the **Delete rdspg-lab-db instance** dialog box:
    - (1) Clear **Create final snapshot**.
    - (2) Clear **Retain automated backups**.
    - (3) Select the acknowledgement check box.
    - (4) Enter `delete me`.
    - (5) Choose **Delete**.

    ![Confirm DB instance deletion](/images/6-cleanup/2.png)

{{% notice info %}}
In a real environment, keep **Create final snapshot** selected so you can restore the data later. In this workshop the data is only sample data, and a snapshot is billed for storage after the DB instance is deleted.
{{% /notice %}}

The DB instance changes to **Deleting** and takes several minutes to disappear. Continue with the EC2 instances in the meantime.

#### Terminate the EC2 instances

1. Open the [Amazon EC2 console](https://console.aws.amazon.com/ec2/):
    - (1) Choose **Instances**.
    - (2) Select `rdspg-app-server` and `rdspg-bastion-host`.
    - (3) Choose **Instance state**.
    - (4) Choose **Terminate (delete) instance**.

    ![Terminate instances](/images/6-cleanup/3.png)

2. In the **Terminate (delete) instances** dialog box, leave **Skip OS shutdown** cleared:
    - (1) Choose **Terminate (delete)**.

    ![Confirm termination](/images/6-cleanup/4.png)

The root EBS volumes are deleted together with the instances.

#### Delete the DB subnet group

Wait until `rdspg-lab-db` no longer appears in **Databases**. A DB subnet group that is still used by a DB instance cannot be deleted.

1. In the Amazon RDS console:
    - (1) Choose **Subnet groups**.
    - (2) Select `rdspg-lab-subnet-group`.
    - (3) Choose **Delete**.

    ![Delete DB subnet group](/images/6-cleanup/5.png)

2. In the confirmation dialog box:
    - (1) Choose **Delete**.

    ![Confirm DB subnet group deletion](/images/6-cleanup/6.png)

#### Delete the security groups

Make sure that both EC2 instances show **Terminated** first. A security group that is still attached to a network interface cannot be deleted.

1. Open the [Amazon VPC console](https://console.aws.amazon.com/vpc/):
    - (1) Choose **Security groups**.
    - (2) Select `bastion-sg`.
    - (3) Select `app-sg` and `rds-sg`.
    - (4) Choose **Actions**.
    - (5) Choose **Delete security groups**.

    ![Delete security groups](/images/6-cleanup/7.png)

2. The dialog box shows that only `rds-sg` will be deleted. `app-sg` is referenced by `rds-sg`, and `bastion-sg` is referenced by `app-sg` and `rds-sg`:
    - (1) Enter `delete`.
    - (2) Choose **Delete**.

    ![Confirm security group deletion](/images/6-cleanup/8.png)

3. Repeat the same steps for `app-sg`, then for `bastion-sg`. Each deletion removes the reference that blocked the next security group.

{{% notice note %}}
This is the same rule you used in chapter 2: a security group can use another security group as the source of an inbound rule. Delete the security groups in the reverse order of the references: `rds-sg` → `app-sg` → `bastion-sg`.
{{% /notice %}}

#### Delete the key pair

1. In the Amazon EC2 console:
    - (1) Choose **Key Pairs**.
    - (2) Select `rdspg-lab-key`.
    - (3) Choose **Actions**.
    - (4) Choose **Delete**.

    ![Delete key pair](/images/6-cleanup/9.png)

2. In the confirmation dialog box:
    - (1) Enter `Delete`.
    - (2) Choose **Delete**.

    ![Confirm key pair deletion](/images/6-cleanup/10.png)

Deleting the key pair in AWS does not delete the `.pem` file on your computer. You delete that file in the last section.

#### Delete the VPC

1. In the Amazon VPC console:
    - (1) Choose **Your VPCs**.
    - (2) Select `rdspg-lab-vpc`.
    - (3) Choose **Actions**.
    - (4) Choose **Delete VPC**.

    ![Delete VPC](/images/6-cleanup/11.png)

2. The dialog box lists the resources that are deleted together with the VPC: the internet gateway, the route tables, the four subnets, and the S3 gateway endpoint.
    - (1) Enter `delete`.
    - (2) Choose **Delete**.

    ![Confirm VPC deletion](/images/6-cleanup/12.png)

#### Clean up your computer

1. In pgAdmin, right-click the `rdspg-lab-db` server and choose **Remove Server**.
2. Delete the `rdspg-bastion` and `rdspg-app` entries from `$HOME\.ssh\config`. If the file contains only these two entries, delete the file.
3. Delete the private key file:

    ```powershell
    Remove-Item "$HOME\.ssh\rdspg-lab-key.pem" -Force
    ```

{{% notice tip %}}
Open the [AWS Billing and Cost Management console](https://console.aws.amazon.com/costmanagement/) the next day and confirm that no new charges for Amazon RDS, Amazon EC2, or public IPv4 addresses appear.
{{% /notice %}}
