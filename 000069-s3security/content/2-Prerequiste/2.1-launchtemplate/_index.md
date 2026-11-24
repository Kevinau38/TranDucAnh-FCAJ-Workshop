---
title : "Deploy the CloudFormation template"
weight : 1
chapter : false
pre : " <b> 2.1 </b> "
---


#### Deploy the CloudFormation template

1. Download the template [fcj-s3-security-lab.yaml](/files/fcj-s3-security-lab.yaml).
2. Open the [CloudFormation console](https://console.aws.amazon.com/cloudformation/home) in the Region you chose, then choose **Create stack** → **With new resources (standard)**.
3. Choose **Upload a template file**, select the downloaded file and choose **Next**.
4. Enter the stack name `FCJ-S3-LAB`.
   + **InstanceType**: keep `t3.micro`.
   + **PermissionsBoundaryArn**: leave empty. Only fill it in if your organization requires a permissions boundary on every IAM role.
5. Choose **Next** twice, select **I acknowledge that AWS CloudFormation might create IAM resources with custom names**, then choose **Submit**.

The template creates:

| Resource | Name |
|---|---|
| VPC, Internet Gateway, public subnet, route table | `SID-vpc`, `SID-public-subnet`, `SID-routes` |
| Security group with **no inbound rules** | `SID-instance-sg` |
| S3 bucket with a test object `app1/file1` | `sid-security-<random>` |
| IAM role and instance profile | `FCJ-S3Security-EC2-Role-<region>` |
| EC2 instance (Amazon Linux 2023) | `SID-security-instance` |

6. Wait until the stack status is **CREATE_COMPLETE** (about 3 minutes), then open the **Outputs** tab. Note the **BucketName** and **VpcId** values.

![Stack outputs](/images/2-prerequisite/2.1-stack-outputs.png)
