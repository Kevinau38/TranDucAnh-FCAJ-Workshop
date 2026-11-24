---
title : "Create the security groups"
weight : 3
chapter : false
pre : " <b> 2.3 </b> "
---

You create three security groups. Each one accepts traffic only from the previous layer:

| Security group | Inbound rule | Used by |
|---|---|---|
| `ALB-SG` | HTTP 80 from the internet (in part 6 you restrict it to CloudFront) | Application Load Balancer |
| `WebServer-SG` | HTTP 80 from `ALB-SG` | WordPress EC2 instances |
| `Database-SG` | MySQL 3306 from `WebServer-SG` | RDS database |

#### Create ALB-SG

1. Open the [Amazon EC2 console](https://console.aws.amazon.com/ec2/) → **Security Groups** → **Create security group**.

2. Enter the following values:
   - (1) **Security group name**: `ALB-SG`, **Description**: `Security group for the internet-facing Application Load Balancer`.
   - (2) **VPC**: `wordpress-vpc`.
   - (3) **Inbound rules**: choose **Add rule**, **Type** `HTTP`, **Source type** `Anywhere-IPv4`, **Description** `HTTP from the internet`.

   ![ALB-SG](/images/2-prerequisite/2.3-alb-sg.png)

3. Keep the default outbound rule and choose **Create security group**.

#### Create WebServer-SG

1. Choose **Create security group** again, select **VPC** `wordpress-vpc` and enter:
   - (1) **Security group name**: `WebServer-SG`, **Description**: `Security group for the private WordPress web servers`.
   - (2) **Inbound rules**: **Type** `HTTP`, **Source** – type `ALB-SG` and select the security group from the list, **Description** `HTTP from the ALB only`.

   ![WebServer-SG](/images/2-prerequisite/2.3-web-sg.png)

2. Choose **Create security group**.

#### Create Database-SG

1. Choose **Create security group** again, select **VPC** `wordpress-vpc` and enter:
   - (1) **Security group name**: `Database-SG`, **Description**: `Security group for the RDS MySQL database`.
   - (2) **Inbound rules**: **Type** `MYSQL/Aurora`, **Source** – select `WebServer-SG`, **Description** `MySQL from the web servers only`.

   ![Database-SG](/images/2-prerequisite/2.3-db-sg.png)

2. Choose **Create security group**.

{{% notice note %}}
Neither `WebServer-SG` nor `Database-SG` has a rule for SSH (port 22). You connect to the instances with Session Manager instead.
{{% /notice %}}
