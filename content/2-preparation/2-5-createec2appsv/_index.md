---
title : "Create EC2 instance for app server"
weight : 5
chapter : false
pre : " <b> 2.5. </b> "
---

#### Overview

In this step, you launch `rdspg-app-server`, an EC2 instance in a **private subnet**. Later, you install the PostgreSQL client (`psql`) on it and connect to Amazon RDS.

| Setting | Value | Reason |
|---|---|---|
| AMI | Amazon Linux 2023 | Amazon Linux 2 reached end of support on June 30, 2026 |
| Instance type | `t3.micro` | Enough for a PostgreSQL client |
| Subnet | `rdspg-lab-subnet-private1-ap-southeast-1a` | The App server does not need to be reachable from the internet |
| Public IP | Disabled | A private instance does not need a public IP address |
| Security group | `app-sg` | Allows SSH only from `bastion-sg` |

You also create the key pair `rdspg-lab-key`, which you will reuse for the bastion host.

#### Launch the instance

1. Open the [Amazon EC2 console](https://console.aws.amazon.com/ec2/):
    - (1) Choose **Instances**.
    - (2) Choose **Launch instances**.

    ![Launch instances](/images/2-preparation/2.5/1.png)

2. Configure the name and image:
    - (1) **Name**: `rdspg-app-server`
    - (2) **Amazon Machine Image (AMI)**: **Amazon Linux 2023 kernel-6.18 AMI**. Keep **Architecture** as **64-bit (x86)**.

    ![Name and AMI](/images/2-preparation/2.5/2.png)

3. Configure the instance type and key pair:
    - (1) **Instance type**: `t3.micro`
    - (2) Choose **Create new key pair**.

    ![Instance type and key pair](/images/2-preparation/2.5/3.png)

4. In **Create key pair**:
    - (1) **Key pair name**: `rdspg-lab-key`
    - (2) **Key pair type**: **RSA**
    - (3) **Private key file format**: **.pem**
    - (4) Choose **Create key pair**.

    ![Create key pair](/images/2-preparation/2.5/4.png)

{{% notice warning %}}
Your browser downloads `rdspg-lab-key.pem`. Store it in a safe location outside the workshop folder. AWS does not let you download the private key again, and you need it to connect to your instances.
{{% /notice %}}

5. In **Network settings**, choose **Edit**:
    - (1) **VPC**: `rdspg-lab-vpc`
    - (2) **Subnet**: `rdspg-lab-subnet-private1-ap-southeast-1a`
    - (3) **Auto-assign public IP**: **Disable**
    - (4) **Firewall (security groups)**: choose **Select existing security group**, then choose `app-sg`.
    - (5) Choose **Launch instance**.

    ![Network settings](/images/2-preparation/2.5/5.png)

{{% notice tip %}}
The subnet shows **4091** available IP addresses. A `/20` subnet has 4,096 addresses, and AWS reserves 5 addresses in every subnet.
{{% /notice %}}

6. The instance is launched.

    ![Launch success](/images/2-preparation/2.5/6.png)

#### Verify the instance

7. Open the instance `rdspg-app-server` and check:
    - (1) **Instance state**: **Running**
    - (2) **Private IPv4 address**: an address in `10.0.128.0/20`
    - (3) **Public IPv4 address**: empty

    ![Instance summary](/images/2-preparation/2.5/7.png)
