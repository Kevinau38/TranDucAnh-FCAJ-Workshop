---
title : "Introduction"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

#### Service orchestration

Orchestration means coordinating distributed services from one central workflow, the way a conductor leads every musician of an orchestra. The logic of the process lives in one place instead of being spread over the services, so you can follow the whole process from start to finish in one view. This works well when steps must be retried, errors must be handled, tasks run in parallel, or the process has to wait for something before it continues.

#### Business domain

The workshop simulates part of a bank. A customer submits an application to open an account, and the application is checked before it is approved.

- **Account Applications** service: stores new applications, flags them for review, approves or rejects them, and lists them by state.
- **Data Checking** service: checks the name and the address of the applicant with simple rules.
- **Accounts** service: opens the account once the application is approved. It is not built in this workshop.

#### Workflow

For every new application the workflow:

1. Checks the name and the address of the applicant.
2. Approves the application automatically if both checks pass. If a check cannot be done, flags the application as unprocessable and stops.
3. Otherwise flags the application for review and pauses.
4. Waits for a reviewer to approve or reject it.
5. Approves or rejects the application according to the decision.

#### Architecture

![Architecture](/images/1-introduction/architecture.png)

- **AWS CloudShell** runs the AWS CLI and the AWS SAM CLI. No IDE or EC2 instance is needed.
- **AWS SAM** packages the code into a private S3 bucket and creates a CloudFormation stack with the Lambda functions, the DynamoDB table, the state machine and two IAM roles.
- `SubmitApplication` stores a new application and starts an execution of the state machine.
- The state machine calls `DataChecking`, `FlagApplication`, `ApproveApplication` and `RejectApplication`. When an application needs a review it waits for a **task token**; `ReviewApplication` sends the decision back with `SendTaskSuccess`.

#### Before you start

- Use the Region **Asia Pacific (Tokyo) ap-northeast-1**.
- If your account requires a **permissions boundary** on new IAM roles, the commands in part 2 pass it to the template. Leave the parameter empty otherwise.

{{% notice warning %}}
Cost: Lambda, Step Functions (Standard), DynamoDB on-demand and S3 are billed per request. Running this workshop costs a few cents. Do part 7 as soon as you finish.
{{% /notice %}}
