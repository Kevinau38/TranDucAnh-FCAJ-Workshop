---
title : "Dọn dẹp tài nguyên"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

{{% notice warning %}}
Hãy xoá tài nguyên ngay khi làm xong: DB instance, secret, EC2 instance và địa chỉ public IPv4 đều tính phí khi còn tồn tại.
{{% /notice %}}

#### Xoá bằng AWS CloudShell

1. Nhấn `Ctrl+C` trong terminal Session Manager để dừng ứng dụng, rồi chọn **Terminate** để kết thúc phiên.

2. Mở [AWS CloudShell](https://console.aws.amazon.com/cloudshell/home) ở Region của workshop, trong cùng thư mục home nơi bạn đã chạy các script ở mục [2.1](../2-preparation/2-1-setupenv/) (các ID nằm trong `~/ids.txt`). Chạy:

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

    Script xoá theo thứ tự:
    - DB instance `rdspg-dev-db`, không tạo final snapshot. **RDS xoá secret `rds!db-<id>` cùng với DB instance**;
    - EC2 instance `rdspg-dev-server`;
    - IAM role và instance profile `FCJ-rdspg-dev-ec2-role`;
    - sau khi các instance đã xoá xong (khoảng 5 tới 10 phút): DB subnet group, các security group, subnet, route table, internet gateway và VPC.

    Dòng cuối in ra `deleted vpc-...`.

#### Kiểm tra

3. Đảm bảo không còn gì sót lại:
    - [RDS console](https://console.aws.amazon.com/rds/home#databases:): không còn `rdspg-dev-db`, và mục **Snapshots** không có manual snapshot nào của nó.
    - [Secrets Manager console](https://console.aws.amazon.com/secretsmanager/listsecrets): không còn secret `rds!db-...` của workshop.
    - [EC2 console](https://console.aws.amazon.com/ec2/home#Instances:): `rdspg-dev-server` ở trạng thái **Terminated**.
    - [VPC console](https://console.aws.amazon.com/vpcconsole/home#vpcs:): không còn `rdspg-dev-vpc`.
    - [IAM console](https://console.aws.amazon.com/iam/home#/roles): không còn `FCJ-rdspg-dev-ec2-role`.
