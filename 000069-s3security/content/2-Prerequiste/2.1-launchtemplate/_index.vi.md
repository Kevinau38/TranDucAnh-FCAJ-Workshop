---
title : "Triển khai CloudFormation template"
weight : 1
chapter : false
pre : " <b> 2.1 </b> "
---


#### Triển khai CloudFormation template

1. Tải template [fcj-s3-security-lab.yaml](/files/fcj-s3-security-lab.yaml).
2. Mở [CloudFormation console](https://console.aws.amazon.com/cloudformation/home) tại Region bạn chọn, chọn **Create stack** → **With new resources (standard)**.
3. Chọn **Upload a template file**, chọn file vừa tải và chọn **Next**.
4. Nhập tên stack `FCJ-S3-LAB`.
   + **InstanceType**: giữ `t3.micro`.
   + **PermissionsBoundaryArn**: để trống. Chỉ điền nếu tổ chức của bạn bắt buộc mọi IAM role phải có permissions boundary.
5. Chọn **Next** hai lần, tích **I acknowledge that AWS CloudFormation might create IAM resources with custom names**, rồi chọn **Submit**.

Template sẽ tạo:

| Tài nguyên | Tên |
|---|---|
| VPC, Internet Gateway, public subnet, route table | `SID-vpc`, `SID-public-subnet`, `SID-routes` |
| Security group **không có inbound rule** | `SID-instance-sg` |
| S3 bucket với object thử nghiệm `app1/file1` | `sid-security-<random>` |
| IAM role và instance profile | `FCJ-S3Security-EC2-Role-<region>` |
| EC2 instance (Amazon Linux 2023) | `SID-security-instance` |

6. Chờ stack chuyển sang **CREATE_COMPLETE** (khoảng 3 phút), mở tab **Outputs** và ghi lại giá trị **BucketName** và **VpcId**.

![Stack outputs](/images/2-prerequisite/2.1-stack-outputs.png)
