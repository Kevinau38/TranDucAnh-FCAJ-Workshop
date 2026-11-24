---
title : "Clean up resources"
weight : 9
chapter : false
pre : " <b> 9. </b> "
---

#### Overview

Delete everything that you created in this workshop, so that you do not pay for it. You run two scripts in AWS CloudShell, in the Region of the workshop:

| Script | Deletes |
|---|---|
| Step 1 (`c1.sh`) | The event subscription, `rdspg-lab-db` with its automated backups, the manual snapshot, the EC2 instances and the key pair, the AWS Backup plan and recovery points, the IAM role `FCJ-rdspg-backup-role`, the CloudWatch alarm, the SNS topic, and the log groups |
| Step 2 (`c2.sh`) | Waits until the DB instance and the EC2 instances are gone, then deletes the backup vault, the parameter group, the DB subnet group, and the whole VPC |

{{% notice warning %}}
The scripts delete resources permanently, without a final snapshot. Check that you no longer need the data. Delete the Multi-AZ DB cluster and its replica first, if you have not done it at the end of step 5.5.
{{% /notice %}}

#### Step 1: Delete the lab resources

1. Open [AWS CloudShell](https://console.aws.amazon.com/cloudshell/), save the script as `c1.sh`, and run `bash c1.sh`:

    ```bash
    # 1. Delete the DB instance, its automated backups, and the manual snapshot
    aws rds delete-event-subscription --subscription-name rdspg-lab-failover --query 'EventSubscription.Status' --output text
    aws rds delete-db-instance --db-instance-identifier rdspg-lab-db \
        --skip-final-snapshot --delete-automated-backups \
        --query 'DBInstance.[DBInstanceIdentifier,DBInstanceStatus]' --output text
    aws rds delete-db-snapshot --db-snapshot-identifier rdspg-lab-db-manual-snapshot \
        --query 'DBSnapshot.[DBSnapshotIdentifier,Status]' --output text
    # 2. Terminate the EC2 instances and delete the key pair
    IDS=$(aws ec2 describe-instances --filters Name=tag:Name,Values=rdspg-bastion-host,rdspg-app-server \
        Name=instance-state-name,Values=running,stopped --query 'Reservations[].Instances[].InstanceId' --output text)
    aws ec2 terminate-instances --instance-ids $IDS --query 'TerminatingInstances[].CurrentState.Name' --output text
    aws ec2 delete-key-pair --key-name rdspg-lab-key
    # 3. Delete AWS Backup: selections, plan, recovery points, vault, and the IAM role
    PLAN=$(aws backup list-backup-plans --query "BackupPlansList[?BackupPlanName=='rdspg-backup-plan'].BackupPlanId" --output text)
    for S in $(aws backup list-backup-selections --backup-plan-id $PLAN --query 'BackupSelectionsList[].SelectionId' --output text); do
        aws backup delete-backup-selection --backup-plan-id $PLAN --selection-id $S; done
    aws backup delete-backup-plan --backup-plan-id $PLAN --query 'BackupPlanId' --output text
    for RP in $(aws backup list-recovery-points-by-backup-vault --backup-vault-name rdspg-backup-vault \
        --query 'RecoveryPoints[].RecoveryPointArn' --output text); do
        aws backup delete-recovery-point --backup-vault-name rdspg-backup-vault --recovery-point-arn $RP; done
    for P in AWSBackupServiceRolePolicyForBackup AWSBackupServiceRolePolicyForRestores; do
        aws iam detach-role-policy --role-name FCJ-rdspg-backup-role \
            --policy-arn arn:aws:iam::aws:policy/service-role/$P; done
    aws iam delete-role --role-name FCJ-rdspg-backup-role
    # 4. Delete monitoring resources
    aws cloudwatch delete-alarms --alarm-names awsrds-rdspg-lab-db-High-CPUUtilization
    TOPIC=$(aws sns list-topics --query "Topics[?ends_with(TopicArn,':rdspg-lab-alarm-topic')].TopicArn" --output text)
    aws sns delete-topic --topic-arn $TOPIC
    for G in $(aws logs describe-log-groups --log-group-name-prefix /aws/rds/instance/rdspg --query 'logGroups[].logGroupName' --output text) \
             $(aws logs describe-log-groups --log-group-name-prefix /aws/rds/cluster/rdspg --query 'logGroups[].logGroupName' --output text); do
        aws logs delete-log-group --log-group-name $G && echo "deleted $G"; done
    ```

    - (1) The event subscription is **deleting**.
    - (2) `rdspg-lab-db` is **deleting**.
    - (3) The manual snapshot is **deleted**.
    - (4) Both EC2 instances are **shutting-down**.
    - (5) The ID of the deleted backup plan.
    - (6) The log groups of the DB instances are deleted.

    ![Clean up step 1](/images/9/9-1/1.png)

#### Step 2: Delete the network and the remaining resources

2. Save the second script as `c2.sh` and run `bash c2.sh`. It waits for the deletions of step 1, which takes a few minutes. `ids.txt` is the file that the network script of the Preparation step created in your CloudShell home directory:

    ```bash
    # Run after rdspg-lab-db is deleted and the EC2 instances are terminated
    source <(tr ' ' '\n' < ~/ids.txt)
    aws rds wait db-instance-deleted --db-instance-identifier rdspg-lab-db
    aws ec2 wait instance-terminated --filters Name=tag:Name,Values=rdspg-bastion-host,rdspg-app-server
    aws backup delete-backup-vault --backup-vault-name rdspg-backup-vault
    aws rds delete-db-parameter-group --db-parameter-group-name rdspg-lab-pg18
    aws rds delete-db-subnet-group --db-subnet-group-name rdspg-lab-subnet-group
    # VPC endpoint, security groups, subnets, route tables, internet gateway, VPC
    aws ec2 delete-vpc-endpoints --vpc-endpoint-ids $(aws ec2 describe-vpc-endpoints \
        --filters Name=vpc-id,Values=$VPC --query 'VpcEndpoints[].VpcEndpointId' --output text) --output text
    for G in $RSG $ASG $BSG; do
        aws ec2 revoke-security-group-ingress --group-id $G --ip-permissions \
            "$(aws ec2 describe-security-groups --group-ids $G --query 'SecurityGroups[0].IpPermissions' --output json)" --output text >/dev/null
    done
    for G in $RSG $ASG $BSG; do aws ec2 delete-security-group --group-id $G; done
    for S in $(aws ec2 describe-subnets --filters Name=vpc-id,Values=$VPC --query 'Subnets[].SubnetId' --output text); do
        aws ec2 delete-subnet --subnet-id $S; done
    for R in $(aws ec2 describe-route-tables --filters Name=vpc-id,Values=$VPC \
        --query 'RouteTables[?!(Associations[?Main])].RouteTableId' --output text); do
        aws ec2 delete-route-table --route-table-id $R; done
    IGW=$(aws ec2 describe-internet-gateways --filters Name=attachment.vpc-id,Values=$VPC \
        --query 'InternetGateways[0].InternetGatewayId' --output text)
    aws ec2 detach-internet-gateway --internet-gateway-id $IGW --vpc-id $VPC
    aws ec2 delete-internet-gateway --internet-gateway-id $IGW
    aws ec2 delete-vpc --vpc-id $VPC && echo "deleted $VPC"
    ```

    The script prints `True` for each deleted security group, then `deleted vpc-...` when the VPC is gone.

#### Check

3. RDS deletes the automated snapshots (`rds:rdspg-lab-db-...` and `rds:preupgrade-...`) of a deleted DB instance by itself when you use `--delete-automated-backups`. It can take some time. Check that nothing is left:

    ```bash
    aws rds describe-db-snapshots \
        --query 'DBSnapshots[?contains(DBSnapshotIdentifier,`rdspg`)].[DBSnapshotIdentifier,Status]' --output text
    aws ec2 describe-vpcs --filters Name=tag:Name,Values=rdspg-lab-vpc --query 'Vpcs[].VpcId' --output text
    ```

    - (1) No snapshot of `rdspg-lab-db` is left.
    - (2) The VPC `rdspg-lab-vpc` no longer exists.

    ![Check the clean-up](/images/9/9-1/2.png)

4. Delete the local files that you no longer need: the key pair `rdspg-lab-key` and the `Host rdspg-*` entries in `~/.ssh/config` on your computer, and `ids.txt`, `c1.sh`, and `c2.sh` in CloudShell.

{{% notice tip %}}
If a command in step 2 fails because a resource is still in use, wait a minute and run `bash c2.sh` again. Resources that are already deleted only print an error such as *NotFound*.
{{% /notice %}}
