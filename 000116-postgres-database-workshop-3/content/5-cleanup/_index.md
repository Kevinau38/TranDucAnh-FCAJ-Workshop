---
title : "Clean up"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

{{% notice warning %}}
Delete the resources as soon as you finish: the DB instance, the secret, the EC2 instance, and its public IPv4 address cost money while they exist.
{{% /notice %}}

#### Delete with AWS CloudShell

1. Press `Ctrl+C` in the Session Manager terminal to stop the application, then choose **Terminate** to end the session.

2. Open [AWS CloudShell](https://console.aws.amazon.com/cloudshell/home) in the Region of the workshop, in the same home folder where you ran the scripts of [2.1](../2-preparation/2-1-setupenv/) (the IDs are in `~/ids.txt`). Run:

    ```bash
    export AWS_DEFAULT_REGION=ap-southeast-1
    source <(tr ' ' '\n' < ~/ids.txt)
    # 1. RDS (the secret that RDS manages is deleted together with the DB instance) and EC2
    aws rds delete-db-instance --db-instance-identifier rdspg-dev-db --skip-final-snapshot \
        --delete-automated-backups --query 'DBInstance.[DBInstanceIdentifier,DBInstanceStatus]' --output text
    EC2=$(aws ec2 describe-instances --filters Name=tag:Name,Values=rdspg-dev-server \
        Name=instance-state-name,Values=running,stopped --query 'Reservations[].Instances[].InstanceId' --output text)
    aws ec2 terminate-instances --instance-ids $EC2 --query 'TerminatingInstances[].CurrentState.Name' --output text
    # 2. IAM role and instance profile
    ROLE=FCJ-rdspg-dev-ec2-role
    aws iam remove-role-from-instance-profile --instance-profile-name $ROLE --role-name $ROLE
    aws iam delete-instance-profile --instance-profile-name $ROLE
    aws iam detach-role-policy --role-name $ROLE --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
    aws iam delete-role-policy --role-name $ROLE --policy-name rdspg-dev-app-access
    aws iam delete-role --role-name $ROLE && echo "deleted $ROLE"
    # 3. Wait, then delete the network
    aws ec2 wait instance-terminated --instance-ids $EC2
    aws rds wait db-instance-deleted --db-instance-identifier rdspg-dev-db
    aws rds delete-db-subnet-group --db-subnet-group-name rdspg-dev-subnet-group
    aws ec2 delete-security-group --group-id $RSG --output text
    aws ec2 delete-security-group --group-id $DSG --output text
    for S in $PUB1 $PRI1 $PRI2; do aws ec2 delete-subnet --subnet-id $S; done
    aws ec2 delete-route-table --route-table-id $RTP
    aws ec2 detach-internet-gateway --internet-gateway-id $IGW --vpc-id $VPC
    aws ec2 delete-internet-gateway --internet-gateway-id $IGW
    aws ec2 delete-vpc --vpc-id $VPC && echo "deleted $VPC"
    ```

    The script deletes, in order:
    - the DB instance `rdspg-dev-db` without a final snapshot. **RDS deletes the secret `rds!db-<id>` together with the DB instance**;
    - the EC2 instance `rdspg-dev-server`;
    - the IAM role and instance profile `FCJ-rdspg-dev-ec2-role`;
    - after the instances are gone (about 5 to 10 minutes): the DB subnet group, the security groups, the subnets, the route table, the internet gateway, and the VPC.

    The last line prints `deleted vpc-...`.

#### Check

3. Make sure nothing is left:
    - [RDS console](https://console.aws.amazon.com/rds/home#databases:): `rdspg-dev-db` is gone, and **Snapshots** has no manual snapshot of it.
    - [Secrets Manager console](https://console.aws.amazon.com/secretsmanager/listsecrets): no `rds!db-...` secret of this workshop.
    - [EC2 console](https://console.aws.amazon.com/ec2/home#Instances:): `rdspg-dev-server` is **Terminated**.
    - [VPC console](https://console.aws.amazon.com/vpcconsole/home#vpcs:): `rdspg-dev-vpc` is gone.
    - [IAM console](https://console.aws.amazon.com/iam/home#/roles): `FCJ-rdspg-dev-ec2-role` is gone.
