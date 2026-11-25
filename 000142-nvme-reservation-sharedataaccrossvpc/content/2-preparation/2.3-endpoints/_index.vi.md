---
title : "Tạo security group và endpoint"
weight : 3
chapter : false
pre : " <b> 2.3 </b> "
---

Các instance không có đường ra Internet nên không gọi được endpoint public của AWS Systems Manager. Ở mỗi VPC, bạn tạo **interface VPC endpoint** cho `ssm`, `ssmmessages`, `ec2messages`, và hai security group:

| Security group | Rule |
|---|---|
| `prod-endpoint-sg`, `test-endpoint-sg` | Inbound HTTPS (443) từ dải CIDR của VPC |
| `prod-instance-sg`, `test-instance-sg` | Không có inbound rule. Rule outbound mặc định cho phép instance gọi tới endpoint |

1. Mở **AWS CloudShell** trên thanh công cụ của console và chạy:

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

   Vòng lặp in ra sáu endpoint ở trạng thái `pending`.

2. Mở VPC console → **Endpoints**. Sau khoảng hai phút:
   - (1) Hai S3 gateway endpoint tạo ở bước 2.1 và 2.2 ở trạng thái **Available**.
   - (2) Sáu interface endpoint ở trạng thái **Available**.

   ![VPC endpoints](/images/2-preparation/2.3-endpoints.png)

{{% notice warning %}}
Interface endpoint tính phí theo giờ ở mỗi Availability Zone. Nhớ xoá ở phần 8.
{{% /notice %}}
