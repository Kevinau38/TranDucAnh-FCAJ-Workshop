---
title : "Chuẩn bị"
weight : 1
chapter : false
pre : " <b> 1.1. </b> "
---

#### Tổng quan

Workshop này cần môi trường giống workshop thứ nhất, với một điểm khác: DB subnet group có private subnet ở Availability Zone **thứ ba**. Nếu bạn vẫn còn môi trường của workshop thứ nhất, chỉ cần thêm subnet thứ ba (bước 2) và kiểm tra phiên bản của `rdspg-lab-db` (bước 5). Nếu không, hãy tạo mọi thứ theo các bước dưới đây, mất khoảng 20 phút.

| Tài nguyên | Tên | Cấu hình |
|---|---|---|
| VPC | `rdspg-lab-vpc` | `10.0.0.0/16`, 2 public subnet, 3 private subnet, internet gateway, S3 gateway endpoint |
| Security group | `bastion-sg`, `app-sg`, `rds-sg` | SSH từ IP của bạn → bastion → App server; PostgreSQL 5432 từ App server và bastion |
| DB subnet group | `rdspg-lab-subnet-group` | 3 private subnet |
| EC2 | `rdspg-bastion-host`, `rdspg-app-server` | Amazon Linux 2023, `t3.micro` |
| RDS | `rdspg-lab-db` | PostgreSQL **17.9**, `db.t4g.micro`, Single-AZ, 20 GiB `gp3` |

{{% notice info %}}
Chương 2 nâng cấp cơ sở dữ liệu từ PostgreSQL 17.9. Nếu Region của bạn không còn bản 17.9, hãy chọn bản 17.x cũ nhất còn có. Các bước giữ nguyên, chỉ khác số phiên bản.
{{% /notice %}}

#### 1. Tìm địa chỉ IP public của bạn

1. Mở [checkip.amazonaws.com](https://checkip.amazonaws.com) và ghi lại địa chỉ. Chỉ địa chỉ này được kết nối vào bastion host.

#### 2. Tạo mạng bằng AWS CloudShell

2. Mở [AWS CloudShell](https://console.aws.amazon.com/cloudshell/) ở Region của bạn. Thay `MYIP` bằng địa chỉ của bạn và kiểm tra ba Availability Zone có tồn tại (`aws ec2 describe-availability-zones --query 'AvailabilityZones[].ZoneName'`). Sau đó chạy:

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

    Dòng cuối lưu các ID vào `ids.txt` trong thư mục home của CloudShell, dùng cho các bước sau và chương Dọn dẹp.

{{% notice tip %}}
Nếu bạn giữ VPC của workshop thứ nhất, chỉ cần thêm private subnet thứ ba: tạo `10.0.160.0/20` ở Availability Zone thứ ba, gắn vào private route table và thêm vào `rdspg-lab-subnet-group` bằng `aws rds modify-db-subnet-group`. Bước 5.4 có sẵn các lệnh.
{{% /notice %}}

#### 3. Tạo key pair và các EC2 instance

3. Trên máy của bạn, tạo key pair nếu chưa có từ workshop thứ nhất: `ssh-keygen -t rsa -b 4096 -f ~/.ssh/rdspg-lab-key`. Sau đó, trong CloudShell, import public key và tạo hai instance. Thay `<your public key>` bằng nội dung của `rdspg-lab-key.pub`:

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

4. Trên máy của bạn, thêm hai host vào `~/.ssh/config` (trên Windows: `C:\Users\<you>\.ssh\config`), với IP public của bastion host và IP private của App server từ kết quả ở trên:

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

    Kiểm tra kết nối bằng `ssh rdspg-app`.

#### 4. Tạo cơ sở dữ liệu RDS

5. Trong [Amazon RDS console](https://console.aws.amazon.com/rds/home#launch-dbinstance:), chọn **Create database** > **Full configuration** > **PostgreSQL**, với các cấu hình dưới đây. Trong **Settings**:
    - (1) **Engine version**: **PostgreSQL 17.9-R2**
    - (2) **DB instance identifier**: `rdspg-lab-db`

    ![Engine version and identifier](/images/1/1-1/1.png)

    Đặt các phần còn lại như sau:

    | Phần | Cấu hình |
    |---|---|
    | Templates | **Free tier** |
    | Credentials Settings | **Master username** `postgres`, **Self managed**, một mật khẩu mạnh mà bạn ghi lại |
    | Instance configuration | **Burstable classes**, `db.t4g.micro` |
    | Storage | `gp3`, 20 GiB; bỏ chọn **Enable storage autoscaling** |
    | Connectivity | **Don't connect to an EC2 compute resource**, VPC `rdspg-lab-vpc`, DB subnet group `rdspg-lab-subnet-group`, **Public access** No, chỉ security group `rds-sg` |
    | Monitoring | **Database Insights - Standard** |
    | Additional configuration | Bỏ chọn **Enable auto minor version upgrade** và **Enable deletion protection**; giữ thời gian lưu backup 1 ngày |

    Chọn **Create database**. DB instance mất khoảng 10 phút.

#### 5. Chuẩn bị App server

6. Kết nối vào App server (`ssh rdspg-app`) và cài bộ công cụ client PostgreSQL 18:

    ```bash
    sudo dnf install -y postgresql18 postgresql18-contrib
    psql --version; pgbench --version
    ```

    - (1) `dnf` cài `postgresql18`, `postgresql18-contrib` và các gói phụ thuộc.
    - (2) `psql` và `pgbench` 18.6 đã được cài. Client mới hơn dùng được với server cũ hơn, nên cùng bộ công cụ này dùng được cả trước và sau khi nâng cấp ở chương 2.

    ![Client tools](/images/1/1-1/2.png)

7. Khi `rdspg-lab-db` ở trạng thái **Available**, sao chép endpoint từ tab **Connectivity & security**. Lưu thông tin kết nối và mật khẩu, rồi kiểm tra phiên bản và tạo cơ sở dữ liệu `pglab`:

    ```bash
    echo 'export PGHOST=<endpoint of rdspg-lab-db> PGUSER=postgres PGSSLMODE=require' >> ~/.bashrc && source ~/.bashrc
    read -s -p "RDS password: " PW && echo "$PGHOST:5432:*:postgres:$PW" > ~/.pgpass && chmod 600 ~/.pgpass && unset PW && echo
    psql -d postgres -c "SELECT version();" -c "CREATE DATABASE pglab;"
    ```

    - (1) Máy chủ chạy **PostgreSQL 17.9**.
    - (2) Cơ sở dữ liệu `pglab` đã được tạo.

    ![Connection settings](/images/1/1-1/3.png)

{{% notice note %}}
`~/.pgpass` chỉ lưu mật khẩu cho endpoint của `rdspg-lab-db`. Khi kết nối tới endpoint khác (read replica hay DB instance được khôi phục), `psql` sẽ hỏi mật khẩu.
{{% /notice %}}
