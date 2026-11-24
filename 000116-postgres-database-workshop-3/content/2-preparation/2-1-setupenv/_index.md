---
title : "Set up the environment"
weight : 1
chapter : false
pre : " <b> 2.1. </b> "
---

#### Overview

You create the network, the IAM role, and the dev server with one script in AWS CloudShell, then the database in the RDS console. Finally, you open a shell on the dev server with Session Manager and install the tools. It takes about 20 minutes, most of it waiting for the database.

#### 1. Create the network, the IAM role, and the dev server

1. Find the public IP address of your computer at [checkip.amazonaws.com](https://checkip.amazonaws.com). Only this address can open the web application in chapter 4.

2. Open [AWS CloudShell](https://console.aws.amazon.com/cloudshell/) in your Region. Replace the value of `MYIP` with your address, then paste and run the script:

    ```bash
    set +H   # keep ! in rds!db-* as text in an interactive shell
    export AWS_DEFAULT_REGION=ap-southeast-1
    MYIP=203.0.113.10/32   # <-- the public IP address of your computer
    t(){ echo "ResourceType=$1,Tags=[{Key=Name,Value=$2}]"; }
    # 1. Network: 1 public subnet for the dev server, 2 private subnets for RDS
    VPC=$(aws ec2 create-vpc --cidr-block 10.0.0.0/16 --tag-specifications "$(t vpc rdspg-dev-vpc)" --query Vpc.VpcId --output text)
    aws ec2 modify-vpc-attribute --vpc-id $VPC --enable-dns-hostnames
    sn(){ aws ec2 create-subnet --vpc-id $VPC --cidr-block $1 --availability-zone $2 --tag-specifications "$(t subnet $3)" --query Subnet.SubnetId --output text; }
    PUB1=$(sn 10.0.0.0/20   ap-southeast-1a rdspg-dev-subnet-public1)
    PRI1=$(sn 10.0.128.0/20 ap-southeast-1a rdspg-dev-subnet-private1)
    PRI2=$(sn 10.0.144.0/20 ap-southeast-1b rdspg-dev-subnet-private2)
    aws ec2 modify-subnet-attribute --subnet-id $PUB1 --map-public-ip-on-launch
    IGW=$(aws ec2 create-internet-gateway --tag-specifications "$(t internet-gateway rdspg-dev-igw)" --query InternetGateway.InternetGatewayId --output text)
    aws ec2 attach-internet-gateway --internet-gateway-id $IGW --vpc-id $VPC
    RTP=$(aws ec2 create-route-table --vpc-id $VPC --tag-specifications "$(t route-table rdspg-dev-rtb-public)" --query RouteTable.RouteTableId --output text)
    aws ec2 create-route --route-table-id $RTP --destination-cidr-block 0.0.0.0/0 --gateway-id $IGW >/dev/null
    aws ec2 associate-route-table --route-table-id $RTP --subnet-id $PUB1 >/dev/null
    # 2. Security groups: the app port only from your IP, PostgreSQL only from the dev server
    sg(){ aws ec2 create-security-group --vpc-id $VPC --group-name $1 --description "$2" --tag-specifications "$(t security-group $1)" --query GroupId --output text; }
    DSG=$(sg dev-server-sg 'Allow the web app port from my IP')
    RSG=$(sg rds-dev-sg 'Allow PostgreSQL from the dev server')
    aws ec2 authorize-security-group-ingress --group-id $DSG --protocol tcp --port 3000 --cidr $MYIP >/dev/null
    aws ec2 authorize-security-group-ingress --group-id $RSG --protocol tcp --port 5432 --source-group $DSG >/dev/null
    aws rds create-db-subnet-group --db-subnet-group-name rdspg-dev-subnet-group \
        --db-subnet-group-description 'Private subnets for rdspg-dev-db' --subnet-ids $PRI1 $PRI2 >/dev/null
    echo "VPC=$VPC PUB1=$PUB1 PRI1=$PRI1 PRI2=$PRI2 IGW=$IGW RTP=$RTP DSG=$DSG RSG=$RSG" | tee ids.txt
    # 3. IAM role for the dev server
    ACCOUNT=$(aws sts get-caller-identity --query Account --output text)
    ROLE=FCJ-rdspg-dev-ec2-role
    aws iam create-role --role-name $ROLE \
        --permissions-boundary arn:aws:iam::$ACCOUNT:policy/LabPermissionsBoundary \
        --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":"ec2.amazonaws.com"},"Action":"sts:AssumeRole"}]}' \
        --query Role.RoleName --output text
    aws iam attach-role-policy --role-name $ROLE --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
    aws iam put-role-policy --role-name $ROLE --policy-name rdspg-dev-app-access --policy-document '{
      "Version": "2012-10-17",
      "Statement": [
        {"Sid": "DescribeDB", "Effect": "Allow", "Action": "rds:DescribeDBInstances",
         "Resource": "arn:aws:rds:ap-southeast-1:'$ACCOUNT':db:rdspg-dev-db"},
        {"Sid": "ReadRdsManagedSecret", "Effect": "Allow", "Action": "secretsmanager:GetSecretValue",
         "Resource": "arn:aws:secretsmanager:ap-southeast-1:'$ACCOUNT':secret:rds!db-*"}
      ]}'
    aws iam create-instance-profile --instance-profile-name $ROLE --query InstanceProfile.InstanceProfileName --output text
    aws iam add-role-to-instance-profile --instance-profile-name $ROLE --role-name $ROLE
    # 4. Dev server: Amazon Linux 2023, no key pair, IMDSv2 only
    AMI=$(aws ssm get-parameter --name /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64 --query Parameter.Value --output text)
    sleep 10   # wait until the new instance profile can be used
    aws ec2 run-instances --image-id $AMI --instance-type t3.micro --subnet-id $PUB1 --security-group-ids $DSG \
        --iam-instance-profile Name=FCJ-rdspg-dev-ec2-role \
        --block-device-mappings 'DeviceName=/dev/xvda,Ebs={VolumeSize=8,VolumeType=gp3,Encrypted=true}' \
        --metadata-options HttpTokens=required \
        --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=rdspg-dev-server}]' \
        --query 'Instances[0].InstanceId' --output text
    ```

    - (1) The script saves the IDs of the network in `ids.txt`, in your CloudShell home directory, for the Clean up chapter.
    - (2) The IAM role and the instance profile `FCJ-rdspg-dev-ec2-role` are created.
    - (3) The ID of the new EC2 instance `rdspg-dev-server`.

    ![CloudShell script](/images/2/2-1/1.png)

{{% notice info %}}
The role follows the principle of least privilege: it can describe only `rdspg-dev-db` and read only the secrets that RDS creates (their names start with `rds!db-`). `AmazonSSMManagedInstanceCore` lets Session Manager connect to the instance. The permissions boundary `LabPermissionsBoundary` is required by the lab account; remove `--permissions-boundary` in your own account.
{{% /notice %}}

#### 2. Create the database

3. Open the [Amazon RDS console](https://console.aws.amazon.com/rds/home#launch-dbinstance:) and choose **Create database**. Choose **Full configuration**, **PostgreSQL**, keep the default **Engine version** (PostgreSQL 18), and choose the **Free tier** template. In **Settings**:
    - (1) **DB instance identifier**: `rdspg-dev-db`
    - (2) **Master username**: `postgres`
    - (3) **Credentials management**: **Managed in AWS Secrets Manager - most secure**. RDS generates the password, stores it in a secret, and nobody has to type or remember it.
    - (4) **Select the encryption key**: keep `aws/secretsmanager (default)`.

    ![Settings](/images/2/2-1/2.png)

4. In **Instance configuration**, keep `db.t4g.micro`. In **Storage**, choose **General Purpose SSD (gp3)**, 20 GiB, and in **Additional storage configuration** clear **Enable storage autoscaling**. In **Connectivity**, keep **Don't connect to an EC2 compute resource** and choose the VPC `rdspg-dev-vpc`. Then:
    - (1) **DB subnet group**: `rdspg-dev-subnet-group`
    - (2) **Public access**: **No**
    - (3) **Existing VPC security groups**: remove `default` and choose `rds-dev-sg`.

    ![Connectivity](/images/2/2-1/3.png)

5. Expand **Additional configuration**:
    - (1) **Initial database name**: `pglab`. RDS creates this database for the application.

    ![Initial database name](/images/2/2-1/4.png)

    Choose **Create database**. The DB instance takes about 10 minutes.

#### 3. Connect to the dev server with Session Manager

6. Open the [Amazon EC2 console](https://console.aws.amazon.com/ec2/home#Instances:), select `rdspg-dev-server`, and choose **Connect**:
    - (1) Choose **SSM Session Manager**.
    - (2) The **Ping status** is **Online** and the instance is **Connected** to Session Manager.
    - (3) The **IAM role** is `FCJ-rdspg-dev-ec2-role`.
    - (4) Choose **Connect**.

    ![Session Manager](/images/2/2-1/5.png)

{{% notice tip %}}
If the ping status is not **Online** yet, wait two or three minutes after the instance starts: the SSM agent needs the IAM role to register with Systems Manager.
{{% /notice %}}

7. A terminal opens in the browser as `ssm-user`. Switch to `ec2-user`, install Node.js 24, Git, and the PostgreSQL 18 client, and save the Region for the AWS SDK:

    ```bash
    sudo su - ec2-user
    sudo dnf install -y -q nodejs24 nodejs24-npm git postgresql18
    node --version; npm --version; git --version; psql --version
    echo 'export AWS_REGION=ap-southeast-1' >> ~/.bashrc && source ~/.bashrc
    aws sts get-caller-identity --query Arn --output text | cut -d: -f6
    ```

    - (1) Node.js **24**, npm **11**, Git, and `psql` **18.6** are installed.
    - (2) The AWS CLI on the server uses the role `FCJ-rdspg-dev-ec2-role` from the instance profile, without access keys.

    ![Tools](/images/2/2-1/6.png)

8. Create the project folder, install the packages, and download the certificate bundle of RDS, which the code uses to verify the server:

    ```bash
    mkdir -p ~/rdspg-dev-app && cd ~/rdspg-dev-app
    cat > package.json <<'EOF'
    {
      "name": "rdspg-dev-app",
      "version": "1.0.0",
      "private": true,
      "scripts": {
        "simple": "node simple-connect.js",
        "auto": "node auto-connect.js",
        "start": "node app.js"
      },
      "dependencies": {
        "@aws-sdk/client-rds": "^3.700.0",
        "@aws-sdk/client-secrets-manager": "^3.700.0",
        "express": "^4.21.0",
        "express-handlebars": "^8.0.0",
        "pg": "^8.13.0"
      }
    }
    EOF
    npm install --no-fund --no-audit
    curl -sSO https://truststore.pki.rds.amazonaws.com/global/global-bundle.pem
    ls
    ```

    `ls` shows `global-bundle.pem`, `node_modules`, `package.json`, and `package-lock.json`.
