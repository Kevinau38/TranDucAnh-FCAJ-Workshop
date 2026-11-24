---
title : "Deploying WordPress on AWS Cloud"
weight : 1
chapter : false
---
# Deploying WordPress on AWS Cloud

### Overview

In this lab you deploy WordPress on AWS with a fully private architecture. The web servers (Amazon EC2) and the database (Amazon RDS for MySQL) run in **private subnets** and have no public IP addresses. The only component that faces the internet is the **Application Load Balancer**, and users reach it through **Amazon CloudFront** over HTTPS.

- The EC2 instances have no internet access (there is no NAT gateway). They download WordPress from an Amazon S3 bucket through an **S3 gateway endpoint**.
- You manage the instances with **AWS Systems Manager Session Manager** through **interface VPC endpoints**, so no SSH port and no bastion host are needed.
- The database password is created and stored by **AWS Secrets Manager**. You never type it.
- An **Auto Scaling group** keeps the web tier available across two Availability Zones, and **Multi-AZ** keeps a standby copy of the database.

![Architecture](/images/architecture.png)

#### Amazon CloudFront
Amazon CloudFront is a content delivery network (CDN) service. In this lab it gives the site an HTTPS address and is the only source that the load balancer accepts traffic from.

#### Application Load Balancer and Target Group
The Application Load Balancer receives HTTP requests and distributes them to the healthy EC2 instances registered in the target group.

#### Auto Scaling Group
An Auto Scaling group is a group of EC2 instances that are launched from the same launch template. It replaces unhealthy instances and can add or remove instances according to a scaling policy.

#### Amazon RDS and Multi-AZ
Amazon RDS is a managed relational database service. With a Multi-AZ deployment, RDS keeps a synchronous standby copy of the database in another Availability Zone and fails over to it automatically.

#### VPC endpoints
VPC endpoints let resources in private subnets reach AWS services without going through the internet. A gateway endpoint is used for Amazon S3, and interface endpoints are used for Systems Manager and Secrets Manager.
