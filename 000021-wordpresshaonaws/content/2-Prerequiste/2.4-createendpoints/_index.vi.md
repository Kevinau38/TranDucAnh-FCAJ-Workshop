---
title : "Tạo các VPC endpoint"
weight : 4
chapter : false
pre : " <b> 2.4 </b> "
---

Các instance private không có route ra internet, vì vậy chúng không thể kết nối đến endpoint public của AWS Systems Manager và AWS Secrets Manager. Bạn tạo các **interface VPC endpoint** cho những dịch vụ này trong các private subnet:

| Endpoint | Mục đích |
|---|---|
| `ssm`, `ssmmessages`, `ec2messages` | Session Manager và Run Command cho các instance |
| `secretsmanager` | Đọc thông tin đăng nhập cơ sở dữ liệu mà RDS lưu trong Secrets Manager |

Các endpoint dùng một security group mới là `Endpoint-SG`, chỉ cho phép HTTPS (443) từ bên trong VPC.

1. Mở **AWS CloudShell** và chạy các câu lệnh sau:

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

2. Mở [Amazon VPC console](https://console.aws.amazon.com/vpc/) → **Endpoints**. Sau khoảng hai phút, bốn interface endpoint và S3 gateway endpoint ở bước 2.2 đều ở trạng thái **Available**.

   ![Các VPC endpoint](/images/2-prerequisite/2.4-endpoints.png)

{{% notice warning %}}
Interface endpoint được tính phí theo giờ và theo từng Availability Zone. Hãy xoá chúng ở bước dọn dẹp khi bạn hoàn thành bài lab.
{{% /notice %}}
