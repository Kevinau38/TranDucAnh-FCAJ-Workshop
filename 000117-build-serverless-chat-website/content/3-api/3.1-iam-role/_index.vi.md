---
title : "Tạo IAM role cho Lambda"
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---

Các Lambda function cần một role có hai quyền: ghi log lên CloudWatch và đọc object trong bucket.

Chạy trong CloudShell:

```bash
echo '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":"lambda.amazonaws.com"},"Action":"sts:AssumeRole"}]}' > lambda-trust.json
aws iam create-role --role-name FCJ-ChatApp-Lambda-Role --assume-role-policy-document file://lambda-trust.json ${BOUNDARY:+--permissions-boundary $BOUNDARY} --query Role.RoleName --output text
aws iam attach-role-policy --role-name FCJ-ChatApp-Lambda-Role --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole && echo ATTACHED
aws iam put-role-policy --role-name FCJ-ChatApp-Lambda-Role --policy-name S3-Access-ChatApp --policy-document "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Action\":\"s3:GetObject\",\"Resource\":\"arn:aws:s3:::$BUCKET/*\"}]}" && echo S3-POLICY-OK
```

- (1) Role `FCJ-ChatApp-Lambda-Role` được tạo với trust policy cho Lambda.
- (2) Managed policy `AWSLambdaBasicExecutionRole` cho phép function ghi log.
- (3) Inline policy `S3-Access-ChatApp` chỉ cho `s3:GetObject` trên đúng bucket này.

![Role cho Lambda](/images/3-api/3.1-lambda-role.png)
