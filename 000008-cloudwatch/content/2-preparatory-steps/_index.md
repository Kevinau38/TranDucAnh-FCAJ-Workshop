---
title : "Preparation"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---


In this section you create the lab environment with one CloudFormation stack.

1. Download the template [fcj-cloudwatch-workshop.yaml](/files/fcj-cloudwatch-workshop.yaml).
2. Open the **CloudFormation** console, choose **Create stack** (1), then **With new resources (standard)**.

![Create stack](/images/2-preparation/2.1-create-stack.png)

3. Keep **Choose an existing template**, select **Upload a template file** (1), choose **Choose file** and select the template you downloaded (2). Choose **Next** (3).

![Upload the template](/images/2-preparation/2.1-upload-template.png)

4. On **Specify stack details**:
   - **Stack name** (1): `FCJ-CloudWatch-Workshop`.
   - **InstanceType** (2): keep `t4g.micro`.
   - **LatestAmiId**: keep the default. It resolves the latest Amazon Linux 2023 arm64 AMI.
   - **PermissionsBoundaryArn** (3): leave it empty in your own account. If your organization requires a permissions boundary for new IAM roles, enter its ARN here. The screenshot uses the boundary required by the FCJ shared account.
   - Choose **Next** (4).

![Stack details](/images/2-preparation/2.1-stack-details.png)

5. On **Configure stack options**, keep the defaults, scroll down, select **I acknowledge that AWS CloudFormation might create IAM resources with custom names** (1) and choose **Next** (2).

![Acknowledge IAM resources](/images/2-preparation/2.1-capabilities.png)

6. Review the settings and choose **Submit** (1).

![Submit](/images/2-preparation/2.1-submit.png)

7. Wait about 3 minutes until the stack status is **CREATE_COMPLETE** (1).

![CREATE_COMPLETE](/images/2-preparation/2.1-create-complete.png)

{{% notice info %}}
After the stack is complete, the instances still need one or two minutes to install the CloudWatch agent and start sending data. Wait about 10 minutes before section 3 so the graphs have some data points.
{{% /notice %}}

The stack creates:

| Resource | Name |
|---|---|
| VPC, public subnet, internet gateway, route table | `FCJ-CloudWatch-VPC`, `FCJ-CloudWatch-Public-Subnet`, ... |
| Security group with no inbound rules | `FCJ-CloudWatch-Instance-SG` |
| IAM role and instance profile | `FCJ-CloudWatch-EC2-Role-<region>` |
| Log group | `/fcj/cloudwatch-workshop/var/log/messages` |
| Three EC2 instances | `FCJ-CloudWatch-Instance-A`, `-B`, `-C` |
