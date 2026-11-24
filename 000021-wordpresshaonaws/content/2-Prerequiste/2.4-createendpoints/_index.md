---
title : "Create the VPC endpoints"
weight : 4
chapter : false
pre : " <b> 2.4 </b> "
---

The private instances have no route to the internet, so they cannot reach the public endpoints of AWS Systems Manager and AWS Secrets Manager. You create **interface VPC endpoints** for these services in the private subnets:

| Endpoint | Purpose |
|---|---|
| `ssm`, `ssmmessages`, `ec2messages` | Session Manager and Run Command for the instances |
| `secretsmanager` | Read the database credentials that RDS stores in Secrets Manager |

The endpoints use a new security group, `Endpoint-SG`, that allows HTTPS (443) only from inside the VPC.

1. Open **AWS CloudShell** and run the following commands:

   ```bash
   VPC=$(aws ec2 describe-vpcs --filters Name=tag:Name,Values=wordpress-vpc --query "Vpcs[0].VpcId" --output text)
   SUBNETS=$(aws ec2 describe-subnets --filters Name=vpc-id,Values=$VPC Name=tag:Name,Values="*private*" --query "Subnets[].SubnetId" --output text)
   ESG=$(aws ec2 create-security-group --group-name Endpoint-SG --description "HTTPS from inside the VPC to the interface endpoints" --vpc-id $VPC --query GroupId --output text)
   aws ec2 authorize-security-group-ingress --group-id $ESG --protocol tcp --port 443 --cidr 10.0.0.0/16 --query Return
   for s in ssm ssmmessages ec2messages secretsmanager; do
     aws ec2 create-vpc-endpoint --vpc-id $VPC --vpc-endpoint-type Interface \
       --service-name com.amazonaws.ap-northeast-1.$s --subnet-ids $SUBNETS \
       --security-group-ids $ESG --private-dns-enabled \
       --tag-specifications "ResourceType=vpc-endpoint,Tags=[{Key=Name,Value=wordpress-$s}]" \
       --query "VpcEndpoint.[ServiceName,State]" --output text
   done
   ```

2. Open the [Amazon VPC console](https://console.aws.amazon.com/vpc/) → **Endpoints**. After about two minutes the four interface endpoints and the S3 gateway endpoint from step 2.2 are **Available**.

   ![VPC endpoints](/images/2-prerequisite/2.4-endpoints.png)

{{% notice warning %}}
Interface endpoints are charged per hour and per Availability Zone. Delete them in the clean-up step when you finish the lab.
{{% /notice %}}
