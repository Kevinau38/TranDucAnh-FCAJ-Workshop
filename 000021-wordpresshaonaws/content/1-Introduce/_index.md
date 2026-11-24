---
title : "Introduction"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

In this lab you deploy a WordPress website that can scale with the number of visitors and keeps working when one Availability Zone fails. Every server is private: the only parts of the architecture that the internet can reach are Amazon CloudFront and, through CloudFront only, the Application Load Balancer.

![Architecture](/images/architecture.png)

#### What you build

| Layer | AWS services | Subnet |
|---|---|---|
| Entry point | Amazon CloudFront (HTTPS) | – |
| Load balancing | Application Load Balancer, target group | Public subnets (ALB only) |
| Web servers | Amazon EC2 in an Auto Scaling group, launch template, AMI | Private subnets |
| Database | Amazon RDS for MySQL 8.4, Multi-AZ, DB snapshot | Private subnets |
| Management | AWS Systems Manager Session Manager, AWS Secrets Manager, Amazon S3 | Reached through VPC endpoints |

#### Security principles of this lab

- **No public subnet for servers.** EC2 and RDS have no public IP addresses. Only the load balancer sits in the public subnets.
- **No NAT gateway and no internet access** for the servers. Installation files come from your own S3 bucket through a gateway endpoint.
- **No SSH and no bastion host.** You manage the instances with Session Manager through interface endpoints.
- **No password typed by hand.** RDS generates the master password and stores it in Secrets Manager; the instance reads it directly.
- **Least privilege between layers.** Each security group accepts traffic only from the previous layer, and the load balancer accepts traffic only from CloudFront.

#### WordPress
WordPress is a free and open-source content management system written in PHP that uses a MySQL or MariaDB database. It is one of the most popular platforms for building blogs, company websites and online stores.

#### Estimated time and cost
The lab takes about 90 minutes. Most of the cost comes from the RDS Multi-AZ instance, the interface endpoints and the load balancer, which are charged per hour. Delete all resources in part 7 as soon as you finish.
