---
title : "Preparation"
weight : 1
chapter : false
pre : " <b> 1.1. </b> "
---

#### Overview

This workshop needs the same environment as the first workshop, with one difference: the DB subnet group has a private subnet in a **third** Availability Zone. If you still have the environment of the first workshop, add the third subnet (step 2) and check the version of `rdspg-lab-db` (step 5). Otherwise, create everything with the steps below. It takes about 20 minutes.

| Resource | Name | Settings |
|---|---|---|
| VPC | `rdspg-lab-vpc` | `10.0.0.0/16`, 2 public subnets, 3 private subnets, internet gateway, S3 gateway endpoint |
| Security groups | `bastion-sg`, `app-sg`, `rds-sg` | SSH from your IP → bastion → App server; PostgreSQL 5432 from App server and bastion |
| DB subnet group | `rdspg-lab-subnet-group` | The 3 private subnets |
| EC2 | `rdspg-bastion-host`, `rdspg-app-server` | Amazon Linux 2023, `t3.micro` |
| RDS | `rdspg-lab-db` | PostgreSQL **17.9**, `db.t4g.micro`, Single-AZ, 20 GiB `gp3` |

{{% notice info %}}
Chapter 2 upgrades the database from PostgreSQL 17.9. If 17.9 is not offered anymore in your Region, choose the oldest available 17.x version. The steps stay the same, only the version numbers are different.
{{% /notice %}}

#### 1. Find your public IP address

1. Open [checkip.amazonaws.com](https://checkip.amazonaws.com) and write down the address. Only this address can connect to the bastion host.

#### 2. Create the network with AWS CloudShell

2. Open [AWS CloudShell](https://console.aws.amazon.com/cloudshell/) in your Region. Replace `MYIP` with your address, and check that the three Availability Zones exist (`aws ec2 describe-availability-zones --query 'AvailabilityZones[].ZoneName'`). Then run:

    ```bash
    MYIP=203.0.113.10/32
    AZ1=ap-southeast-1a; AZ2=ap-southeast-1b; AZ3=ap-southeast-1c
    t(){ echo "ResourceType=$1,Tags=[{Key=Name,Value=$2}]"; }
    VPC=$(aws ec2 create-vpc --cidr-block 10.0.0.0/16 --tag-specifications "$(t vpc rdspg-lab-vpc)" --query Vpc.VpcId --output text)
    aws ec2 modify-vpc-attribute --vpc-id $VPC --enable-dns-hostnames
    sn(){ aws ec2 create-subnet --vpc-id $VPC --cidr-block $1 --availability-zone $2 --tag-specifications "$(t subnet $3)" --query Subnet.SubnetId --output text; }
    PUB1=$(sn 10.0.0.0/20   $AZ1 rdspg-lab-subnet-public1)
    PUB2=$(sn 10.0.16.0/20  $AZ2 rdspg-lab-subnet-public2)
    PRI1=$(sn 10.0.128.0/20 $AZ1 rdspg-lab-subnet-private1)
    PRI2=$(sn 10.0.144.0/20 $AZ2 rdspg-lab-subnet-private2)
    PRI3=$(sn 10.0.160.0/20 $AZ3 rdspg-lab-subnet-private3)
    for s in $PUB1 $PUB2; do aws ec2 modify-subnet-attribute --subnet-id $s --map-public-ip-on-launch; done
    IGW=$(aws ec2 create-internet-gateway --tag-specifications "$(t internet-gateway rdspg-lab-igw)" --query InternetGateway.InternetGatewayId --output text)
    aws ec2 attach-internet-gateway --internet-gateway-id $IGW --vpc-id $VPC
    RTP=$(aws ec2 create-route-table --vpc-id $VPC --tag-specifications "$(t route-table rdspg-lab-rtb-public)" --query RouteTable.RouteTableId --output text)
    RTI=$(aws ec2 create-route-table --vpc-id $VPC --tag-specifications "$(t route-table rdspg-lab-rtb-private)" --query RouteTable.RouteTableId --output text)
    aws ec2 create-route --route-table-id $RTP --destination-cidr-block 0.0.0.0/0 --gateway-id $IGW >/dev/null
    for s in $PUB1 $PUB2; do aws ec2 associate-route-table --route-table-id $RTP --subnet-id $s >/dev/null; done
    for s in $PRI1 $PRI2 $PRI3; do aws ec2 associate-route-table --route-table-id $RTI --subnet-id $s >/dev/null; done
    aws ec2 create-vpc-endpoint --vpc-id $VPC --service-name com.amazonaws.${AZ1%?}.s3 --route-table-ids $RTI \
        --tag-specifications "$(t vpc-endpoint rdspg-lab-vpce-s3)" >/dev/null
    sg(){ aws ec2 create-security-group --vpc-id $VPC --group-name $1 --description "$2" --tag-specifications "$(t security-group $1)" --query GroupId --output text; }
    BSG=$(sg bastion-sg 'Allow SSH from my IP to bastion host')
    ASG=$(sg app-sg 'Allow SSH from bastion host')
    RSG=$(sg rds-sg 'Allow PostgreSQL from app server and bastion host')
    aws ec2 authorize-security-group-ingress --group-id $BSG --protocol tcp --port 22 --cidr $MYIP >/dev/null
    aws ec2 authorize-security-group-ingress --group-id $ASG --protocol tcp --port 22 --source-group $BSG >/dev/null
    aws ec2 authorize-security-group-ingress --group-id $RSG --protocol tcp --port 5432 --source-group $ASG >/dev/null
    aws ec2 authorize-security-group-ingress --group-id $RSG --protocol tcp --port 5432 --source-group $BSG >/dev/null
    aws rds create-db-subnet-group --db-subnet-group-name rdspg-lab-subnet-group \
        --db-subnet-group-description 'Private subnets for rdspg-lab-db' --subnet-ids $PRI1 $PRI2 $PRI3 >/dev/null
    echo "VPC=$VPC PUB1=$PUB1 PRI1=$PRI1 BSG=$BSG ASG=$ASG RSG=$RSG" | tee ids.txt
    ```

    The last line saves the IDs in `ids.txt`, in your CloudShell home directory, for the next steps and for the Clean up chapter.

{{% notice tip %}}
If you kept the VPC of the first workshop, add only the third private subnet: create `10.0.160.0/20` in a third Availability Zone, associate it with the private route table, and add it to `rdspg-lab-subnet-group` with `aws rds modify-db-subnet-group`. Step 5.4 shows the commands.
{{% /notice %}}

#### 3. Create the key pair and the EC2 instances

3. On your computer, create a key pair if you do not have one from the first workshop: `ssh-keygen -t rsa -b 4096 -f ~/.ssh/rdspg-lab-key`. Then, in CloudShell, import the public key and launch the two instances. Replace `<your public key>` with the content of `rdspg-lab-key.pub`:

    ```bash
    source <(tr ' ' '\n' < ids.txt)
    aws ec2 import-key-pair --key-name rdspg-lab-key --public-key-material fileb://<(echo '<your public key>') >/dev/null
    AMI=$(aws ssm get-parameter --name /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64 --query Parameter.Value --output text)
    run(){ aws ec2 run-instances --image-id $AMI --instance-type t3.micro --key-name rdspg-lab-key \
        --subnet-id $1 --security-group-ids $2 \
        --block-device-mappings 'DeviceName=/dev/xvda,Ebs={VolumeSize=8,VolumeType=gp3,Encrypted=true}' \
        --metadata-options HttpTokens=required \
        --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=$3}]" \
        --query Instances[0].InstanceId --output text; }
    B=$(run $PUB1 $BSG rdspg-bastion-host); A=$(run $PRI1 $ASG rdspg-app-server)
    aws ec2 wait instance-running --instance-ids $B $A
    aws ec2 describe-instances --instance-ids $B $A \
        --query 'Reservations[].Instances[].[Tags[?Key==`Name`]|[0].Value,PublicIpAddress,PrivateIpAddress]' --output text
    ```

4. On your computer, add the two hosts to `~/.ssh/config` (on Windows: `C:\Users\<you>\.ssh\config`), with the public IP address of the bastion host and the private IP address of the App server from the output above:

    ```
    Host rdspg-bastion
        HostName <public IP of rdspg-bastion-host>
        User ec2-user
        IdentityFile ~/.ssh/rdspg-lab-key

    Host rdspg-app
        HostName <private IP of rdspg-app-server>
        User ec2-user
        IdentityFile ~/.ssh/rdspg-lab-key
        ProxyJump rdspg-bastion
    ```

    Check the connection with `ssh rdspg-app`.

#### 4. Create the RDS database

5. In the [Amazon RDS console](https://console.aws.amazon.com/rds/home#launch-dbinstance:), choose **Create database** > **Full configuration** > **PostgreSQL**, with the settings below. In **Settings**:
    - (1) **Engine version**: **PostgreSQL 17.9-R2**
    - (2) **DB instance identifier**: `rdspg-lab-db`

    ![Engine version and identifier](/images/1/1-1/1.png)

    Set the other sections as follows:

    | Section | Setting |
    |---|---|
    | Templates | **Free tier** |
    | Credentials Settings | **Master username** `postgres`, **Self managed**, a strong password that you keep |
    | Instance configuration | **Burstable classes**, `db.t4g.micro` |
    | Storage | `gp3`, 20 GiB; clear **Enable storage autoscaling** |
    | Connectivity | **Don't connect to an EC2 compute resource**, VPC `rdspg-lab-vpc`, DB subnet group `rdspg-lab-subnet-group`, **Public access** No, security group `rds-sg` only |
    | Monitoring | **Database Insights - Standard** |
    | Additional configuration | Clear **Enable auto minor version upgrade** and **Enable deletion protection**; keep the backup retention of 1 day |

    Choose **Create database**. The DB instance takes about 10 minutes.

#### 5. Prepare the App server

6. Connect to the App server (`ssh rdspg-app`) and install the PostgreSQL 18 client tools:

    ```bash
    sudo dnf install -y postgresql18 postgresql18-contrib
    psql --version; pgbench --version
    ```

    - (1) `dnf` installs `postgresql18`, `postgresql18-contrib`, and their dependencies.
    - (2) `psql` and `pgbench` 18.6 are installed. A newer client works with an older server, so the same tools work before and after the upgrade in chapter 2.

    ![Client tools](/images/1/1-1/2.png)

7. When `rdspg-lab-db` is **Available**, copy its endpoint from the **Connectivity & security** tab. Save the connection settings and the password, then check the version and create the database `pglab`:

    ```bash
    echo 'export PGHOST=<endpoint of rdspg-lab-db> PGUSER=postgres PGSSLMODE=require' >> ~/.bashrc && source ~/.bashrc
    read -s -p "RDS password: " PW && echo "$PGHOST:5432:*:postgres:$PW" > ~/.pgpass && chmod 600 ~/.pgpass && unset PW && echo
    psql -d postgres -c "SELECT version();" -c "CREATE DATABASE pglab;"
    ```

    - (1) The server runs **PostgreSQL 17.9**.
    - (2) The database `pglab` is created.

    ![Connection settings](/images/1/1-1/3.png)

{{% notice note %}}
`~/.pgpass` stores the password for the endpoint of `rdspg-lab-db` only. When you connect to another endpoint (a read replica or a restored DB instance), `psql` asks for the password.
{{% /notice %}}
