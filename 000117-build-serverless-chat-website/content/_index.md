---
title : "Build a Complete Serverless Chat Website"
weight : 1
chapter : false
---

# Build a Complete Serverless Chat Website

#### Overview

In this workshop you build a small chat website without managing any server. You start with static HTML files and grow the application step by step until users can sign up, sign in and chat with each other.

- The website files live in a **private Amazon S3 bucket** and reach users through **Amazon CloudFront**.
- **Amazon API Gateway** and **AWS Lambda** handle the requests from the browser.
- **Amazon DynamoDB** stores conversations and messages.
- **Amazon Cognito** manages users and protects the API.

#### Content

The workshop has eight parts. Follow them in order: every part reuses the resources of the previous one.

{{% notice info %}}
This is an updated version of the workshop. The bucket is never made public: the site is served through CloudFront with Origin Access Control. Lambda uses a current Node.js runtime, the IAM roles follow the `FCJ-*` naming rule with a permissions boundary, and every step was run again in **Asia Pacific (Tokyo) ap-northeast-1**.
{{% /notice %}}
