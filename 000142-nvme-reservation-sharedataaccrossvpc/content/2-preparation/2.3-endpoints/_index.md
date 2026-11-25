---
title : "Create security groups and endpoints"
weight : 3
chapter : false
pre : " <b> 2.3 </b> "
---

The instances have no route to the internet, so they cannot reach the public endpoints of AWS Systems Manager. In each VPC you create **interface VPC endpoints** for `ssm`, `ssmmessages` and `ec2messages`, and two security groups:

| Security group | Rules |
|---|---|
| `prod-endpoint-sg`, `test-endpoint-sg` | Inbound HTTPS (443) from the CIDR of the VPC |
| `prod-instance-sg`, `test-instance-sg` | No inbound rule. The default outbound rule lets the instance reach the endpoints |

1. Open **AWS CloudShell** from the console toolbar and run:

   ```bash
   for E in prod test; do
     VPC=$(aws ec2 describe-vpcs --filters Name=tag:Name,Values=$E-vpc --query "Vpcs[0].VpcId" --output text)
     CIDR=$(aws ec2 describe-vpcs --vpc-ids $VPC --query "Vpcs[0].CidrBlock" --output text)
     SUBNET=$(aws ec2 describe-subnets --filters Name=vpc-id,Values=$VPC --query "Subnets[0].SubnetId" --output text)
     ESG=$(aws ec2 create-security-group --group-name $E-endpoint-sg --description "HTTPS from inside the $E VPC" --vpc-id $VPC --query GroupId --output text)
     aws ec2 authorize-security-group-ingress --group-id $ESG --protocol tcp --port 443 --cidr $CIDR > /dev/null
     aws ec2 create-security-group --group-name $E-instance-sg --description "$E instance, no inbound rules" --vpc-id $VPC --query GroupId --output text > /dev/null
     for s in ssm ssmmessages ec2messages; do
       aws ec2 create-vpc-endpoint --vpc-id $VPC --vpc-endpoint-type Interface --service-name com.amazonaws.ap-northeast-1.$s --subnet-ids $SUBNET --security-group-ids $ESG --private-dns-enabled --tag-specifications "ResourceType=vpc-endpoint,Tags=[{Key=Name,Value=$E-$s}]" --query "VpcEndpoint.[Tags[0].Value,VpcEndpointId,State]" --output text
     done
   done
   ```

   The loop prints six endpoints in the `pending` state.

2. Open the VPC console → **Endpoints**. After about two minutes:
   - (1) The two S3 gateway endpoints from steps 2.1 and 2.2 are **Available**.
   - (2) The six interface endpoints are **Available**.

   ![VPC endpoints](/images/2-preparation/2.3-endpoints.png)

{{% notice warning %}}
Interface endpoints are charged per hour in each Availability Zone. Delete them in part 8.
{{% /notice %}}
