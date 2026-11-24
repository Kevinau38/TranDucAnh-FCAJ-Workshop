---
title : "Create the RDS database"
weight : 7
chapter : false
pre : " <b> 2.7 </b> "
---

You create an Amazon RDS for MySQL 8.4 database in the private subnets:

| Setting | Value | Why |
|---|---|---|
| DB subnet group | `wordpress-db-subnet-group` (the two private subnets) | The database is never in a public subnet |
| Multi-AZ | Yes | RDS keeps a synchronous standby in the other Availability Zone |
| Public access | No | Only `WebServer-SG` can connect, through `Database-SG` |
| Master password | `--manage-master-user-password` | RDS generates the password and stores it in AWS Secrets Manager, so nobody types or copies it |
| Initial database | `wordpress` | WordPress uses this database |
| Enhanced Monitoring | Off (`--monitoring-interval 0`) | Enhanced Monitoring needs an extra IAM role |

1. In **AWS CloudShell**, run:

   ```bash
   VPC=$(aws ec2 describe-vpcs --filters Name=tag:Name,Values=wordpress-vpc --query "Vpcs[0].VpcId" --output text)
   SUBNETS=$(aws ec2 describe-subnets --filters Name=vpc-id,Values=$VPC Name=tag:Name,Values="*private*" --query "Subnets[].SubnetId" --output text)
   DBSG=$(aws ec2 describe-security-groups --filters Name=vpc-id,Values=$VPC Name=group-name,Values=Database-SG --query "SecurityGroups[0].GroupId" --output text)
   aws rds create-db-subnet-group --db-subnet-group-name wordpress-db-subnet-group \
     --db-subnet-group-description "Private subnets for the WordPress database" \
     --subnet-ids $SUBNETS --query DBSubnetGroup.SubnetGroupStatus --output text
   aws rds create-db-instance --db-instance-identifier wordpress-db \
     --engine mysql --engine-version 8.4.11 --db-instance-class db.t4g.micro \
     --allocated-storage 20 --storage-type gp3 --storage-encrypted \
     --master-username admin --manage-master-user-password \
     --multi-az --db-subnet-group-name wordpress-db-subnet-group \
     --vpc-security-group-ids $DBSG --db-name wordpress \
     --backup-retention-period 1 --no-publicly-accessible \
     --monitoring-interval 0 --no-deletion-protection \
     --query "DBInstance.[DBInstanceIdentifier,DBInstanceStatus,MultiAZ]" --output text
   ```

2. Open the [Amazon RDS console](https://console.aws.amazon.com/rds/) → **Databases** → `wordpress-db`. Creating a Multi-AZ instance takes about 15 minutes. You can continue with part 3 while the status is **Creating**; the WordPress configuration step needs the database to have an endpoint.

3. When the status is **Available**, open the **Configuration** tab and check (1) the engine version `8.4.x`, (2) the database name `wordpress`, (3) **Multi-AZ** = **Yes** and (4) **Enhanced Monitoring** = **Disabled**.

   ![RDS database](/images/2-prerequisite/2.7-rds.png)
