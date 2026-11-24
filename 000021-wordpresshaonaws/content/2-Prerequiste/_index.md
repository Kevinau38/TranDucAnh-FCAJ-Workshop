---
title : "Preparation"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

In this part you prepare everything that WordPress needs before it is installed:

- An S3 bucket with the WordPress installation files, because the web servers have no internet access.
- A VPC with public subnets (used only by the load balancer) and private subnets (used by EC2 and RDS), without a NAT gateway.
- Three security groups that form a chain: **internet → ALB-SG → WebServer-SG → Database-SG**.
- Interface VPC endpoints so that the private instances can use Session Manager and Secrets Manager.
- An IAM role for the EC2 instances.
- A private EC2 instance and an RDS for MySQL database with Multi-AZ.

{{% notice info %}}
This lab uses the **Asia Pacific (Tokyo) ap-northeast-1** Region. If you use another Region, replace `ap-northeast-1` in the commands with your Region code.
{{% /notice %}}

To learn more about VPC, EC2 and RDS basics, you can refer to these labs:
  - [Working with Amazon VPC](https://000003.awsfcaj.com/)
  - [Amazon EC2](https://000004.awsfcaj.com/)
  - [Amazon RDS](https://000005.awsfcaj.com/)
