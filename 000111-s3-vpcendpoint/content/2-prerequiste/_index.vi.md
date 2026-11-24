---
title : "Các bước chuẩn bị"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---


#### Yêu cầu

+ Một tài khoản AWS có quyền với **CloudFormation, EC2/VPC, IAM (tạo role và instance profile), S3, Systems Manager và Route 53 / Route 53 Resolver**.
+ Region bạn chọn còn chỗ cho **thêm 2 VPC** và 1 Internet Gateway (quota mặc định là 5 VPC mỗi Region).
+ Chi phí ước tính: khoảng **0,5–1 USD** nếu hoàn thành và dọn dẹp trong vòng 2 giờ (interface endpoint, Resolver endpoint và hai EC2 t3.micro tính phí theo giờ).

#### Tải CloudFormation template

| Template | Dùng ở | Tạo ra |
|---|---|---|
| [fcj-s3vpce-base.yaml](/files/fcj-s3vpce-base.yaml) | trang này | VPC Cloud, VPC On-prem, VPC peering, route table, security group, 3 SSM interface endpoint, IAM role + instance profile, 2 EC2 instance |
| [fcj-s3vpce-dns.yaml](/files/fcj-s3vpce-dns.yaml) | mục 4.1 | Private hosted zone cho S3, Route 53 Resolver inbound và outbound endpoint |

{{% notice note %}}
Template base có tham số tùy chọn **PermissionsBoundaryArn**. Để trống nếu dùng tài khoản cá nhân. Nếu tổ chức của bạn bắt buộc gắn permissions boundary cho IAM role mới, hãy nhập ARN của boundary đó.
{{% /notice %}}

#### Triển khai stack base

1. Mở **AWS CloudFormation** console tại Region bạn chọn, chọn **Create stack → With new resources (standard)**.
2. Chọn **Upload a template file**, chọn file **fcj-s3vpce-base.yaml**, rồi **Next**.
3. Stack name: **fcj-s3vpce-base**. Giữ nguyên tham số mặc định và chọn **Next** hai lần.
4. Đánh dấu **I acknowledge that AWS CloudFormation might create IAM resources with custom names**, rồi **Submit**.
5. Đợi khoảng **5 phút** đến khi trạng thái là **CREATE_COMPLETE**. Tab **Resources** liệt kê 27 tài nguyên đã được tạo.

![base stack](/images/2-prerequisite/2.1-base-stack.png)

6. Mở **VPC console → Endpoints**. Bạn sẽ thấy ba interface endpoint (**ssm**, **ssmmessages**, **ec2messages**). Chúng giúp Session Manager kết nối tới instance private trong VPC Cloud vốn không có Internet.

![ssm endpoints](/images/2-prerequisite/2.2-ssm-endpoints.png)
