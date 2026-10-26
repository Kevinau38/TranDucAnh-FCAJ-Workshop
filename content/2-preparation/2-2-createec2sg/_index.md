---
title : "Create EC2 Security Groups"
weight : 2
chapter : false
pre : " <b> 2.2. </b> "
---

#### Overview

In this step, you create two security groups for the EC2 instances:

| Security group | Attached to | Inbound rule |
|---|---|---|
| `bastion-sg` | Bastion host | SSH (22) from **My IP** |
| `app-sg` | App server | SSH (22) from **bastion-sg** |

Only your computer can reach the bastion host, and only the bastion host can reach the App server.

{{% notice info %}}
`app-sg` uses another **security group as the source** instead of an IP address. Any instance that has `bastion-sg` attached is allowed, even if its IP address changes.
{{% /notice %}}

#### Create bastion-sg

1. Open the [Amazon EC2 console](https://console.aws.amazon.com/ec2/), choose **Security Groups** under **Network & Security**, then choose **Create security group**.

    ![Create security group](/images/2-preparation/2.2/1.png)

2. In **Basic details**:
    - (1) **Security group name**: `bastion-sg`
    - (2) **Description**: `Allow SSH from my IP to bastion host`
    - (3) **VPC**: `rdspg-lab-vpc`

    ![bastion-sg basic details](/images/2-preparation/2.2/2.png)

{{% notice warning %}}
The **VPC** field defaults to your default VPC. Make sure you choose `rdspg-lab-vpc`. The security group name and description cannot be changed after creation.
{{% /notice %}}

3. In **Inbound rules**, choose **Add rule**: (1) **Type**: `SSH`, (2) **Source type**: `My IP`.

    ![bastion-sg inbound rule](/images/2-preparation/2.2/3.png)

4. (1) Keep the default **Outbound rules**. (2) Choose **Create security group**.

    ![bastion-sg outbound rule](/images/2-preparation/2.2/4.png)

5. `bastion-sg` is created.

    ![bastion-sg created](/images/2-preparation/2.2/5.png)

{{% notice tip %}}
Your public IP address may change. If you later cannot connect to the bastion host over SSH, edit this inbound rule and choose **My IP** again.
{{% /notice %}}

#### Create app-sg

6. Choose **Create security group** again. In **Basic details**:
    - (1) **Security group name**: `app-sg`
    - (2) **Description**: `Allow SSH from bastion host to app server`
    - (3) **VPC**: `rdspg-lab-vpc`

    ![app-sg basic details](/images/2-preparation/2.2/6.png)

7. In **Inbound rules**, choose **Add rule**: (1) **Type**: `SSH`, (2) **Source type**: `Custom`, (3) **Source**: choose `bastion-sg`.

    ![app-sg inbound rule](/images/2-preparation/2.2/7.png)

8. Keep the default outbound rules and choose **Create security group**. The inbound rule of `app-sg` allows SSH only from `bastion-sg`.

    ![app-sg created](/images/2-preparation/2.2/8.png)
