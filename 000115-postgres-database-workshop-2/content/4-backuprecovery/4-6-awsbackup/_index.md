---
title : "AWS Backup"
weight : 6
chapter : false
pre : " <b> 4.6. </b> "
---

#### Overview

**AWS Backup** manages backups of many AWS services (RDS, Aurora, DynamoDB, EBS, EFS, S3 and more) from one place. Instead of configuring each database, you write a **backup plan** once and assign resources to it, for example every RDS database with a given tag.

| Component | Name in this lab | Purpose |
|---|---|---|
| IAM role | `FCJ-rdspg-backup-role` | Lets AWS Backup create and restore recovery points |
| Backup vault | `rdspg-backup-vault` | Stores and encrypts the recovery points |
| On-demand backup | — | A single backup, started now |
| Backup plan | `rdspg-backup-plan`, rule `rdspg-daily` | Daily backup at 00:30, kept 7 days |
| Resource assignment | `rdspg-tagged-resources` | Every RDS database with the tag `backup-plan` = `rdspg-daily` |

#### Create the IAM role

AWS Backup can create a default role for you. In this lab you create your own role, so that you see exactly which permissions AWS Backup gets.

1. Open the [IAM console](https://console.aws.amazon.com/iam/home#/roles), choose **Roles** > **Create role**:
    - (1) **Trusted entity type**: **AWS service**
    - (2) **Service or use case**: **AWS Backup**
    - (3) **Use case**: **AWS Backup**
    - (4) Choose **Next**.

    ![Trusted entity](/images/4/4-6/1.png)

2. Enter `AWSBackupServiceRolePolicyFor` in the search box:
    - (1) The list shows the AWS managed policies for AWS Backup.
    - (2) Select `AWSBackupServiceRolePolicyForBackup`.
    - (3) Select `AWSBackupServiceRolePolicyForRestores`.

    Choose **Next**.

    ![Add permissions](/images/4/4-6/2.png)

3. On **Name, review, and create**:
    - (1) **Role name**: `FCJ-rdspg-backup-role`

    ![Role name](/images/4/4-6/3.png)

4. Scroll down:
    - (1) Check that the two policies are attached as **Permissions policy**.
    - (2) Choose **Create role**.

    ![Create role](/images/4/4-6/4.png)

{{% notice note %}}
If your AWS account requires a **permissions boundary** for new roles (like the shared account used to write this lab), set it in **Set permissions boundary** on the **Add permissions** page. It then appears as **Boundary policy** in the summary.
{{% /notice %}}

#### Create a backup vault

5. Open the [AWS Backup console](https://console.aws.amazon.com/backup/home#/backupvaults) in the Region of your database. Choose **Vaults** > **Create new vault**:
    - (1) **Vault name**: `rdspg-backup-vault`
    - (2) **Vault type**: **Backup vault**
    - (3) **Encryption key**: `(default) aws/backup`

    Choose **Create vault** at the bottom of the page.

    ![Create vault](/images/4/4-6/5.png)

#### Run an on-demand backup

6. Choose **Protected resources** > **Create on-demand backup**:
    - (1) **Resource type**: **RDS**
    - (2) **Database name**: `rdspg-lab-db`
    - (3) **Backup window**: **Create backup now**
    - (4) **Total retention period**: `1` Days

    ![On-demand backup settings](/images/4/4-6/6.png)

7. Scroll down:
    - (1) **Backup vault**: `rdspg-backup-vault`
    - (2) **IAM role**: **Use another role**
    - (3) Choose `FCJ-rdspg-backup-role`.
    - (4) Choose **Create on-demand backup**.

    ![On-demand backup vault and role](/images/4/4-6/7.png)

{{% notice info %}}
The console warns that a retention period shorter than 7 days can cause additional charges. This applies to resource types with a minimum storage duration. For this lab, 1 day is enough, and you delete the recovery point in the Clean up step.
{{% /notice %}}

8. AWS Backup opens **Jobs**. After about 10 minutes, the job is **Completed**. Choose **Vaults** > `rdspg-backup-vault`:
    - (1) The recovery point of `rdspg-lab-db` is **Completed**, with the backup type **Snapshot**.

    ![Recovery point](/images/4/4-6/8.png)

#### Create a backup plan

9. Choose **Backup plans** > **Create backup plan**:
    - (1) **Start options**: **Build a new plan**
    - (2) **Backup plan name**: `rdspg-backup-plan`
    - (3) **Backup rule name**: `rdspg-daily`

    ![Backup plan](/images/4/4-6/9.png)

10. Continue in the backup rule:
    - (1) **Backup vault**: `rdspg-backup-vault`
    - (2) **Backup frequency**: **Daily**
    - (3) **Start time**: keep the default `00:30` in your time zone.

    ![Backup rule schedule](/images/4/4-6/10.png)

11. Scroll down:
    - (1) Keep **Enable continuous backups for point-in-time recovery (PITR)** cleared. The automated backups of RDS already give you point-in-time recovery.
    - (2) **Total retention period**: `7` Days

    Choose **Create plan** at the bottom of the page.

    ![Backup rule retention](/images/4/4-6/11.png)

#### Assign resources by tag

12. AWS Backup opens **Assign resources**:
    - (1) **Resource assignment name**: `rdspg-tagged-resources`
    - (2) **IAM role**: **Use another role**
    - (3) Choose `FCJ-rdspg-backup-role`.

    ![Assign resources](/images/4/4-6/12.png)

13. Scroll down:
    - (1) **Define resource selection**: **Include specific resource types**
    - (2) **Resource type**: **RDS**, **Database names**: **All databases**
    - (3) **Refine selection using tags**: **Key** `backup-plan`, **Condition for value** **Equals**, **Value** `rdspg-daily`

    Choose **Assign resources**, then **Continue** in the confirmation box.

    ![Resource selection](/images/4/4-6/13.png)

{{% notice tip %}}
The tag condition is important. Without it, the plan backs up **every** RDS database in the account and the Region.
{{% /notice %}}

14. The backup plan is ready:
    - (1) The rule `rdspg-daily` stores the backups in `rdspg-backup-vault`.
    - (2) The resource assignment `rdspg-tagged-resources` uses `FCJ-rdspg-backup-role`.

    ![Backup plan details](/images/4/4-6/14.png)

15. Add the tag to the database. In the RDS console, choose `rdspg-lab-db` > **Tags** > **Manage tags** > **Add new tag**:
    - (1) **Key**: `backup-plan`
    - (2) **Value**: `rdspg-daily`
    - (3) Choose **Save changes**.

    ![Tag the database](/images/4/4-6/15.png)

From now on, AWS Backup backs up `rdspg-lab-db` every day at 00:30. To add another database to the plan, you only add the same tag.

#### (Optional) AWS CLI

{{%expand "Start an on-demand backup with the AWS CLI" %}}
```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
DB_ARN=$(aws rds describe-db-instances --db-instance-identifier rdspg-lab-db \
    --query 'DBInstances[0].DBInstanceArn' --output text)

aws backup create-backup-vault --backup-vault-name rdspg-backup-vault

aws backup start-backup-job \
    --backup-vault-name rdspg-backup-vault \
    --resource-arn $DB_ARN \
    --iam-role-arn arn:aws:iam::$ACCOUNT_ID:role/FCJ-rdspg-backup-role \
    --lifecycle DeleteAfterDays=1

aws backup list-backup-jobs --by-backup-vault-name rdspg-backup-vault \
    --query 'BackupJobs[].[State,PercentDone,CreationDate]' --output table
```
{{% /expand %}}
