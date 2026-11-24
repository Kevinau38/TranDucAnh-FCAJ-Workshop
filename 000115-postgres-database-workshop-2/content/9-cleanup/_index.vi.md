---
title : "Dọn dẹp tài nguyên"
weight : 9
chapter : false
pre : " <b> 9. </b> "
---

#### Tổng quan

Xoá toàn bộ tài nguyên đã tạo trong workshop để không phát sinh chi phí. Bạn chạy hai script trong AWS CloudShell, ở Region của workshop:

| Script | Xoá |
|---|---|
| Bước 1 (`c1.sh`) | Event subscription, `rdspg-lab-db` cùng automated backup, manual snapshot, các EC2 instance và key pair, backup plan và recovery point của AWS Backup, IAM role `FCJ-rdspg-backup-role`, CloudWatch alarm, SNS topic và các log group |
| Bước 2 (`c2.sh`) | Chờ DB instance và các EC2 instance bị xoá xong, rồi xoá backup vault, parameter group, DB subnet group và toàn bộ VPC |

{{% notice warning %}}
Các script xoá tài nguyên vĩnh viễn, không tạo final snapshot. Hãy chắc chắn bạn không còn cần dữ liệu. Nếu chưa xoá Multi-AZ DB cluster và replica của nó ở cuối bước 5.5, hãy xoá chúng trước.
{{% /notice %}}

#### Bước 1: Xoá tài nguyên của lab

1. Mở [AWS CloudShell](https://console.aws.amazon.com/cloudshell/), lưu script thành `c1.sh` rồi chạy `bash c1.sh`:

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

    - (1) Event subscription đang **deleting**.
    - (2) `rdspg-lab-db` đang **deleting**.
    - (3) Manual snapshot đã **deleted**.
    - (4) Hai EC2 instance đang **shutting-down**.
    - (5) ID của backup plan đã xoá.
    - (6) Các log group của DB instance đã được xoá.

    ![Dọn dẹp bước 1](/images/9/9-1/1.png)

#### Bước 2: Xoá mạng và các tài nguyên còn lại

2. Lưu script thứ hai thành `c2.sh` rồi chạy `bash c2.sh`. Script chờ các thao tác xoá của bước 1, mất vài phút. `ids.txt` là file do script tạo mạng ở bước Preparation tạo ra trong thư mục home của CloudShell:

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

    Script in `True` cho mỗi security group đã xoá, rồi `deleted vpc-...` khi VPC đã được xoá.

#### Kiểm tra

3. Khi dùng `--delete-automated-backups`, RDS tự xoá các automated snapshot (`rds:rdspg-lab-db-...` và `rds:preupgrade-...`) của DB instance đã xoá. Việc này có thể mất một lúc. Kiểm tra không còn gì sót lại:

    ```bash
    aws rds describe-db-snapshots \
        --query 'DBSnapshots[?contains(DBSnapshotIdentifier,`rdspg`)].[DBSnapshotIdentifier,Status]' --output text
    aws ec2 describe-vpcs --filters Name=tag:Name,Values=rdspg-lab-vpc --query 'Vpcs[].VpcId' --output text
    ```

    - (1) Không còn snapshot nào của `rdspg-lab-db`.
    - (2) VPC `rdspg-lab-vpc` không còn tồn tại.

    ![Kiểm tra dọn dẹp](/images/9/9-1/2.png)

4. Xoá các file cục bộ không còn dùng: key pair `rdspg-lab-key` và các mục `Host rdspg-*` trong `~/.ssh/config` trên máy bạn, cùng `ids.txt`, `c1.sh` và `c2.sh` trong CloudShell.

{{% notice tip %}}
Nếu một lệnh ở bước 2 lỗi vì tài nguyên vẫn đang được dùng, hãy chờ một phút rồi chạy lại `bash c2.sh`. Tài nguyên đã xoá trước đó chỉ in ra lỗi như *NotFound*.
{{% /notice %}}
