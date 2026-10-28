---
title : "Create bastion host EC2 instance"
weight : 6
chapter : false
pre : " <b> 2.6. </b> "
---

#### Overview

In this step, you launch `rdspg-bastion-host`, an EC2 instance in a **public subnet**. It is the only instance that you connect to from your computer. From the bastion host, you reach the App server and Amazon RDS in the private subnets.

The bastion host uses the same settings as the App server in step 2.5, except for the following:

| Setting | App server | Bastion host |
|---|---|---|
| Name | `rdspg-app-server` | `rdspg-bastion-host` |
| Subnet | Private (`10.0.128.0/20`) | **Public (`10.0.0.0/20`)** |
| Auto-assign public IP | Disable | **Enable** |
| Security group | `app-sg` | **`bastion-sg`** |
| Key pair | Create `rdspg-lab-key` | **Reuse `rdspg-lab-key`** |

{{% notice info %}}
The bastion host needs a public IP address so that you can connect to it over SSH from your computer. It stays secure because `bastion-sg` allows SSH only from your IP address.
{{% /notice %}}

#### Launch the instance

1. In the [Amazon EC2 console](https://console.aws.amazon.com/ec2/):
    - (1) Choose **Instances**.
    - (2) Choose **Launch instances**.

    ![Launch instances](/images/2-preparation/2.6/1.png)

2. Configure the name and image:
    - (1) **Name**: `rdspg-bastion-host`
    - (2) **Amazon Machine Image (AMI)**: **Amazon Linux 2023 kernel-6.18 AMI**

    ![Name and AMI](/images/2-preparation/2.6/2.png)

3. Configure the instance type and key pair:
    - (1) **Instance type**: `t3.micro`
    - (2) **Key pair name**: choose the existing key pair `rdspg-lab-key`.

    ![Instance type and key pair](/images/2-preparation/2.6/3.png)

4. In **Network settings**, choose **Edit**:
    - (1) **VPC**: `rdspg-lab-vpc`
    - (2) **Subnet**: `rdspg-lab-subnet-public1-ap-southeast-1a`
    - (3) **Auto-assign public IP**: **Enable**
    - (4) **Firewall (security groups)**: choose **Select existing security group**, then choose `bastion-sg`.
    - (5) Choose **Launch instance**.

    ![Network settings](/images/2-preparation/2.6/4.png)

5. The instance is launched.

    ![Launch success](/images/2-preparation/2.6/5.png)

#### Verify the instance

6. Open the instance `rdspg-bastion-host` and check:
    - **Instance state**: **Running**
    - **Public IPv4 address**: has a value
    - **Private IPv4 address**: an address in `10.0.0.0/20`

    ![Instance summary](/images/2-preparation/2.6/6.png)

{{% notice tip %}}
An auto-assigned public IP address changes every time you stop and start the instance. If you cannot connect over SSH later, check the current public IP address of the bastion host.
{{% /notice %}}
