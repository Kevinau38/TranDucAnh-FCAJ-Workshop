---
title : "Workflow Orchestration with AWS Step Functions"
weight : 1
chapter : false
---

# Workflow Orchestration with AWS Step Functions

#### Overview

In this workshop you build a small bank account application process and orchestrate it with **AWS Step Functions**. The services are simple **AWS Lambda** functions that store data in **Amazon DynamoDB**. You start with a state machine that does nothing, then add tasks, permissions, data flow, branching, a human review step, error handling and parallel work.

- **AWS Step Functions** runs the workflow as a state machine written in Amazon States Language.
- **AWS Lambda** implements the Account Applications and Data Checking services (Node.js, AWS SDK for JavaScript v3).
- **Amazon DynamoDB** stores the applications.
- **AWS SAM** deploys everything from a template, run from **AWS CloudShell**.

#### Content

The workshop has seven parts. Follow them in order: every part changes the application built in the previous one.

{{% notice info %}}
This is an updated version of the workshop. It uses AWS CloudShell instead of AWS Cloud9, Lambda runs Node.js 24 with the AWS SDK for JavaScript v3, every IAM role is declared with a `FCJ-*` name and a permissions boundary, and all steps were run again in **Asia Pacific (Tokyo) ap-northeast-1**.
{{% /notice %}}
