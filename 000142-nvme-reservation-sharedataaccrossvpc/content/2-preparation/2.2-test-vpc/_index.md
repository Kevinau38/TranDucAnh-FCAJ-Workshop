---
title : "Create the test VPC"
weight : 2
chapter : false
pre : " <b> 2.2 </b> "
---

1. In the VPC console, choose **Create VPC** again and use the same settings as step 2.1, except:
   - Name: `test`.
   - **IPv4 CIDR block**: `10.20.0.0/16`.

2. Check the settings before you create the VPC:
   - (1) **NAT gateways**: **None**.
   - (2) **VPC endpoints**: **S3 Gateway**.
   - (3) The preview shows `test-subnet-private1-ap-northeast-1a`, in the same zone as the production subnet.
   - (4) Choose **Create VPC**.

   ![NAT gateway and S3 endpoint](/images/2-preparation/2.2-vpc-nat-endpoint.png)

3. Open **Subnets**. (1) You have one private subnet in each VPC, `prod-subnet-private1-ap-northeast-1a` in `prod-vpc` and `test-subnet-private1-ap-northeast-1a` in `test-vpc`.

   ![Private subnets](/images/2-preparation/2.2-subnets.png)
