---
title : "Create the IAM role for Lambda"
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---

The Lambda functions need a role with two permissions: write logs to CloudWatch, and read objects from the bucket.

Run in CloudShell:

```bash
echo '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":"lambda.amazonaws.com"},"Action":"sts:AssumeRole"}]}' > lambda-trust.json
aws iam create-role --role-name FCJ-ChatApp-Lambda-Role --assume-role-policy-document file://lambda-trust.json ${BOUNDARY:+--permissions-boundary $BOUNDARY} --query Role.RoleName --output text
aws iam attach-role-policy --role-name FCJ-ChatApp-Lambda-Role --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole && echo ATTACHED
aws iam put-role-policy --role-name FCJ-ChatApp-Lambda-Role --policy-name S3-Access-ChatApp --policy-document "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Action\":\"s3:GetObject\",\"Resource\":\"arn:aws:s3:::$BUCKET/*\"}]}" && echo S3-POLICY-OK
```

- (1) The role `FCJ-ChatApp-Lambda-Role` is created with a trust policy for Lambda.
- (2) The managed policy `AWSLambdaBasicExecutionRole` lets the functions write logs.
- (3) The inline policy `S3-Access-ChatApp` allows `s3:GetObject` on the bucket only.

![Lambda role](/images/3-api/3.1-lambda-role.png)
