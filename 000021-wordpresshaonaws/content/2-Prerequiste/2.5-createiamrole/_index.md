---
title : "Create the IAM role for EC2"
weight : 5
chapter : false
pre : " <b> 2.5 </b> "
---

The WordPress instances need an IAM role with three kinds of permissions:

- **AmazonSSMManagedInstanceCore** (AWS managed policy): lets Session Manager and Run Command manage the instance.
- **secretsmanager:GetSecretValue** on the RDS-managed secrets (`rds!*`): lets the instance read the database user name and password when WordPress is configured.
- **s3:GetObject** on the bucket from step 2.1: lets the instance download WordPress and WP-CLI.

{{% notice info %}}
The lab account requires every IAM role name to start with `FCJ-` and to use the permissions boundary `LabPermissionsBoundary`. If your own account has no permissions boundary, remove the `--permissions-boundary` line.
{{% /notice %}}

1. In **AWS CloudShell**, run the following commands. Replace the bucket name with your own:

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

2. The output ends with `ROLE-OK`. Open the [IAM console](https://console.aws.amazon.com/iam/) → **Roles** → (1) `FCJ-WordPress-EC2-Role`. The (2) **Permissions** tab shows the AWS managed policy and the inline policy `WordPressAppAccess`.

   ![IAM role](/images/2-prerequisite/2.5-role.png)
