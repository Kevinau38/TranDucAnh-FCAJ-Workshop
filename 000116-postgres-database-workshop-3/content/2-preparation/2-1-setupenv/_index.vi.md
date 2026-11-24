---
title : "Thiết lập môi trường"
weight : 1
chapter : false
pre : " <b> 2.1. </b> "
---

#### Tổng quan

Bạn tạo mạng, IAM role và dev server bằng một script trong AWS CloudShell, rồi tạo database trong RDS console. Cuối cùng, bạn mở shell trên dev server bằng Session Manager và cài công cụ. Mất khoảng 20 phút, phần lớn là chờ database.

#### 1. Tạo mạng, IAM role và dev server

1. Xem địa chỉ IP public của máy bạn tại [checkip.amazonaws.com](https://checkip.amazonaws.com). Chỉ địa chỉ này mở được ứng dụng web ở chương 4.

2. Mở [AWS CloudShell](https://console.aws.amazon.com/cloudshell/) ở Region của bạn. Thay giá trị `MYIP` bằng địa chỉ của bạn, rồi dán và chạy script:

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

    - (1) Script lưu ID của mạng vào `ids.txt` trong thư mục home của CloudShell, dùng cho chương Dọn dẹp.
    - (2) IAM role và instance profile `FCJ-rdspg-dev-ec2-role` đã được tạo.
    - (3) ID của EC2 instance mới `rdspg-dev-server`.

    ![Script CloudShell](/images/2/2-1/1.png)

{{% notice info %}}
Role tuân theo nguyên tắc quyền tối thiểu: chỉ được describe `rdspg-dev-db` và chỉ đọc được các secret do RDS tạo (tên bắt đầu bằng `rds!db-`). `AmazonSSMManagedInstanceCore` cho phép Session Manager kết nối tới instance. Permissions boundary `LabPermissionsBoundary` là yêu cầu của tài khoản lab; hãy bỏ `--permissions-boundary` khi làm trên tài khoản của bạn.
{{% /notice %}}

#### 2. Tạo database

3. Mở [Amazon RDS console](https://console.aws.amazon.com/rds/home#launch-dbinstance:) và chọn **Create database**. Chọn **Full configuration**, **PostgreSQL**, giữ **Engine version** mặc định (PostgreSQL 18), và chọn template **Free tier**. Ở phần **Settings**:
    - (1) **DB instance identifier**: `rdspg-dev-db`
    - (2) **Master username**: `postgres`
    - (3) **Credentials management**: **Managed in AWS Secrets Manager - most secure**. RDS tự sinh mật khẩu, lưu vào secret, và không ai phải gõ hay nhớ mật khẩu.
    - (4) **Select the encryption key**: giữ `aws/secretsmanager (default)`.

    ![Settings](/images/2/2-1/2.png)

4. Ở **Instance configuration**, giữ `db.t4g.micro`. Ở **Storage**, chọn **General Purpose SSD (gp3)**, 20 GiB, và trong **Additional storage configuration** bỏ chọn **Enable storage autoscaling**. Ở **Connectivity**, giữ **Don't connect to an EC2 compute resource** và chọn VPC `rdspg-dev-vpc`. Sau đó:
    - (1) **DB subnet group**: `rdspg-dev-subnet-group`
    - (2) **Public access**: **No**
    - (3) **Existing VPC security groups**: bỏ `default` và chọn `rds-dev-sg`.

    ![Connectivity](/images/2/2-1/3.png)

5. Mở rộng **Additional configuration**:
    - (1) **Initial database name**: `pglab`. RDS tạo sẵn database này cho ứng dụng.

    ![Initial database name](/images/2/2-1/4.png)

    Chọn **Create database**. DB instance mất khoảng 10 phút để tạo.

#### 3. Kết nối tới dev server bằng Session Manager

6. Mở [Amazon EC2 console](https://console.aws.amazon.com/ec2/home#Instances:), chọn `rdspg-dev-server`, rồi chọn **Connect**:
    - (1) Chọn **SSM Session Manager**.
    - (2) **Ping status** là **Online** và instance đã **Connected** với Session Manager.
    - (3) **IAM role** là `FCJ-rdspg-dev-ec2-role`.
    - (4) Chọn **Connect**.

    ![Session Manager](/images/2/2-1/5.png)

{{% notice tip %}}
Nếu ping status chưa **Online**, hãy chờ hai, ba phút sau khi instance khởi động: SSM agent cần IAM role để đăng ký với Systems Manager.
{{% /notice %}}

7. Một terminal mở trong trình duyệt với user `ssm-user`. Chuyển sang `ec2-user`, cài Node.js 24, Git, PostgreSQL 18 client, và lưu Region cho AWS SDK:

    ```bash
    sudo su - ec2-user
    sudo dnf install -y -q nodejs24 nodejs24-npm git postgresql18
    node --version; npm --version; git --version; psql --version
    echo 'export AWS_REGION=ap-southeast-1' >> ~/.bashrc && source ~/.bashrc
    aws sts get-caller-identity --query Arn --output text | cut -d: -f6
    ```

    - (1) Node.js **24**, npm **11**, Git và `psql` **18.6** đã được cài.
    - (2) AWS CLI trên server dùng role `FCJ-rdspg-dev-ec2-role` từ instance profile, không cần access key.

    ![Công cụ](/images/2/2-1/6.png)

8. Tạo thư mục dự án, cài các gói, và tải bộ chứng chỉ của RDS mà code dùng để xác minh server:

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

    `ls` hiển thị `global-bundle.pem`, `node_modules`, `package.json` và `package-lock.json`.
