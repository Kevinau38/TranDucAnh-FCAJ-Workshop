---
title : "Introduction"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

The chat application has a static front end and a serverless back end. The browser downloads HTML, CSS and JavaScript from CloudFront, then calls a REST API to read and write messages.

#### Architecture

![Architecture](/images/1-introduction/architecture.png)

- **Amazon S3** stores the website files. The bucket blocks all public access.
- **Amazon CloudFront** delivers the files over HTTPS. It reads the bucket through **Origin Access Control (OAC)**, so the bucket only trusts this distribution.
- **Amazon API Gateway** exposes the REST API `Chat-API`. At the end of the workshop a **Cognito authorizer** checks the ID token of every request.
- **AWS Lambda** runs one small function per API method.
- **Amazon DynamoDB** keeps two tables: `Chat-Conversations` (who is in which conversation) and `Chat-Messages` (the messages).
- **Amazon Cognito** stores the users. The website signs users up and in with the Cognito JavaScript library.

#### Source code

The website and the Lambda code come from the repository [HoangGuruu/ws2-chatapp](https://github.com/HoangGuruu/ws2-chatapp). It has one folder per step (`v1` to `v15`). You download it in CloudShell in part 2.

#### Before you start

- Use the Region **Asia Pacific (Tokyo) ap-northeast-1** for all regional resources.
- You run most commands in **AWS CloudShell**. Open it from the icon on the console navigation bar.
- If your account requires a **permissions boundary** on new IAM roles, set it in CloudShell before part 3:

  ```bash
  BOUNDARY=arn:aws:iam::<account-id>:policy/<boundary-name>
  ```

  Leave it empty otherwise. The commands add `--permissions-boundary` only when `BOUNDARY` is set.

{{% notice warning %}}
Cost: every service in this workshop is billed per request or per GB. A few hours of testing costs a few cents. Do part 8 as soon as you finish.
{{% /notice %}}
