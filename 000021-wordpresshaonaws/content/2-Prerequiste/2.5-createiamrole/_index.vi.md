---
title : "Tạo IAM role cho EC2"
weight : 5
chapter : false
pre : " <b> 2.5 </b> "
---

Các instance WordPress cần một IAM role với ba nhóm quyền:

- **AmazonSSMManagedInstanceCore** (AWS managed policy): cho phép Session Manager và Run Command quản lý instance.
- **secretsmanager:GetSecretValue** trên các secret do RDS quản lý (`rds!*`): cho phép instance đọc tên đăng nhập và mật khẩu cơ sở dữ liệu khi cấu hình WordPress.
- **s3:GetObject** trên bucket ở bước 2.1: cho phép instance tải WordPress và WP-CLI.

{{% notice info %}}
Tài khoản lab yêu cầu tên IAM role phải bắt đầu bằng `FCJ-` và phải dùng permissions boundary `LabPermissionsBoundary`. Nếu tài khoản của bạn không có permissions boundary, hãy bỏ dòng `--permissions-boundary`.
{{% /notice %}}

1. Trong **AWS CloudShell**, chạy các câu lệnh sau. Thay tên bucket bằng bucket của bạn:

   ```bash
   ACCOUNT=$(aws sts get-caller-identity --query Account --output text)
   BUCKET=fcj-wordpress-files-1795416336
   R=FCJ-WordPress-EC2-Role
   aws iam create-role --role-name $R \
     --permissions-boundary arn:aws:iam::$ACCOUNT:policy/LabPermissionsBoundary \
     --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":"ec2.amazonaws.com"},"Action":"sts:AssumeRole"}]}' \
     --query Role.RoleName --output text
   aws iam attach-role-policy --role-name $R --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
   aws iam put-role-policy --role-name $R --policy-name WordPressAppAccess --policy-document '{
     "Version": "2012-10-17",
     "Statement": [
       {"Effect": "Allow", "Action": "secretsmanager:GetSecretValue",
        "Resource": "arn:aws:secretsmanager:ap-northeast-1:'$ACCOUNT':secret:rds!*"},
       {"Effect": "Allow", "Action": "rds:DescribeDBInstances", "Resource": "*"},
       {"Effect": "Allow", "Action": "s3:GetObject", "Resource": "arn:aws:s3:::'$BUCKET'/*"}
     ]}'
   aws iam create-instance-profile --instance-profile-name $R --query InstanceProfile.InstanceProfileName --output text
   aws iam add-role-to-instance-profile --instance-profile-name $R --role-name $R && echo ROLE-OK
   ```

2. Kết quả kết thúc bằng `ROLE-OK`. Mở [IAM console](https://console.aws.amazon.com/iam/) → **Roles** → (1) `FCJ-WordPress-EC2-Role`. Tab (2) **Permissions** hiển thị AWS managed policy và inline policy `WordPressAppAccess`.

   ![IAM role](/images/2-prerequisite/2.5-role.png)
