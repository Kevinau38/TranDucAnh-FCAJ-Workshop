---
title : "Prerequisites"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---


#### Requirements

+ An AWS account with permissions for **CloudFormation, EC2/VPC, IAM (create role and instance profile), S3, Systems Manager and Route 53 / Route 53 Resolver**.
+ Room for **two more VPCs** and one Internet Gateway in the Region you choose (the default quota is 5 VPCs per Region).
+ Estimated cost: about **US$0.5–1** if you finish and clean up within 2 hours (interface endpoints, Resolver endpoints and two t3.micro instances are billed per hour).

#### Download the CloudFormation templates

| Template | Used in | Creates |
|---|---|---|
| [fcj-s3vpce-base.yaml](/files/fcj-s3vpce-base.yaml) | this page | VPC Cloud, VPC On-prem, VPC peering, route tables, security groups, 3 SSM interface endpoints, IAM role + instance profile, 2 EC2 instances |
| [fcj-s3vpce-dns.yaml](/files/fcj-s3vpce-dns.yaml) | section 4.1 | Private hosted zone for S3, Route 53 Resolver inbound and outbound endpoints |

{{% notice note %}}
The base template has an optional parameter **PermissionsBoundaryArn**. Leave it empty in your own account. If your organisation requires a permissions boundary on new IAM roles, enter its ARN.
{{% /notice %}}

#### Deploy the base stack

1. Open the **AWS CloudFormation** console in your chosen Region, choose **Create stack → With new resources (standard)**.
2. Choose **Upload a template file**, select **fcj-s3vpce-base.yaml**, then **Next**.
3. Stack name: **fcj-s3vpce-base**. Keep the default parameters and choose **Next** twice.
4. Tick **I acknowledge that AWS CloudFormation might create IAM resources with custom names**, then **Submit**.
5. Wait about **5 minutes** until the status is **CREATE_COMPLETE**. The **Resources** tab lists the 27 resources that were created.

![base stack](/images/2-prerequisite/2.1-base-stack.png)

6. Open the **VPC console → Endpoints**. You will see three interface endpoints (**ssm**, **ssmmessages**, **ec2messages**). They let Session Manager reach the private instance in VPC Cloud, which has no Internet access.

![ssm endpoints](/images/2-prerequisite/2.2-ssm-endpoints.png)
