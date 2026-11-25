---
title : "Pause the workflow for a review"
weight : 2
chapter : false
pre : " <b> 4.2 </b> "
---

Step Functions can integrate with a service in three ways:

- **Request response**: call the service and move on as soon as it answers. The Data Checking tasks work this way.
- **Run a job** (`.sync`): call the service and wait until the job is finished. Not used here.
- **Wait for a callback** (`.waitForTaskToken`): call the service with a **task token**, then pause until someone returns that token with `SendTaskSuccess` or `SendTaskFailure`.

The review uses the callback pattern:

- **Pending Review** calls `FlagApplication` through `arn:aws:states:::lambda:invoke.waitForTaskToken` and passes the token `$$.Task.Token`. The function stores the token with the application.
- A new function `ReviewApplication` takes an ID and a decision, updates the application and calls `SendTaskSuccess` with the stored token and `{"decision": ...}`.
- A new Choice state **Review Approved?** reads `$.review.decision` and goes to **Approve Application** or **Reject Application**.

1. Replace `flag.js` (it now stores `taskToken`) and create `review.js`:

```bash
cd ~/workshop-dir
cat > functions/account-applications/flag.js <<'EOF'
const AccountApplications = require('./AccountApplications');

exports.handler = async (event) => {
    const { id, flagType, taskToken } = event;
    if (flagType === 'REVIEW') {
        return AccountApplications.update(id, { state: 'FLAGGED_FOR_REVIEW', reason: event.reason, taskToken });
    }
    if (flagType === 'UNPROCESSABLE_DATA') {
        const reason = JSON.parse(event.errorInfo.Cause).errorMessage;
        return AccountApplications.update(id, { state: 'FLAGGED_WITH_UNPROCESSABLE_DATA', reason });
    }
    throw new Error('flagType must be REVIEW or UNPROCESSABLE_DATA');
};
EOF
cat > functions/account-applications/review.js <<'EOF'
const { SFNClient, SendTaskSuccessCommand } = require('@aws-sdk/client-sfn');
const AccountApplications = require('./AccountApplications');

const sfn = new SFNClient({});

exports.handler = async (event) => {
    const { id, decision } = event;
    if (decision !== 'APPROVE' && decision !== 'REJECT') {
        throw new Error("decision must be 'APPROVE' or 'REJECT'");
    }
    const state = decision === 'APPROVE' ? 'REVIEW_APPROVED' : 'REVIEW_REJECTED';
    const application = await AccountApplications.update(id, { state });
    await sfn.send(new SendTaskSuccessCommand({
        taskToken: application.taskToken,
        output: JSON.stringify({ decision })
    }));
    return application;
};
EOF
```

2. Replace `template.yaml`. It adds `ReviewApplicationFunction`, allows `states:SendTaskSuccess` and `states:SendTaskFailure` in `StateMachineAccess`, lets the state machine role invoke `FlagApplicationFunction`, and passes `FlagApplicationFunctionName` to the definition:

```bash
cat > template.yaml <<'EOF'
AWSTemplateFormatVersion: "2010-09-09"
Transform: AWS::Serverless-2016-10-31
Description: Template for step-functions-workshop

Parameters:
  PermissionsBoundaryArn:
    Type: String
    Default: ""

Conditions:
  HasBoundary: !Not [!Equals [!Ref PermissionsBoundaryArn, ""]]

Globals:
  Function:
    Runtime: nodejs24.x
    Timeout: 10

Resources:
  FunctionsRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: FCJ-SFN-Functions-Role
      PermissionsBoundary: !If [HasBoundary, !Ref PermissionsBoundaryArn, !Ref AWS::NoValue]
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal: { Service: lambda.amazonaws.com }
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
      Policies:
        - PolicyName: ApplicationsTableAccess
          PolicyDocument:
            Version: "2012-10-17"
            Statement:
              - Effect: Allow
                Action: [dynamodb:GetItem, dynamodb:PutItem, dynamodb:DeleteItem, dynamodb:Query]
                Resource:
                  - !GetAtt ApplicationsTable.Arn
                  - !Sub "${ApplicationsTable.Arn}/index/*"
        - PolicyName: StateMachineAccess
          PolicyDocument:
            Version: "2012-10-17"
            Statement:
              - Effect: Allow
                Action: [states:StartExecution, states:SendTaskSuccess, states:SendTaskFailure]
                Resource: !Sub "arn:aws:states:${AWS::Region}:${AWS::AccountId}:stateMachine:ApplicationProcessingStateMachine"

  StateMachineRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: FCJ-SFN-StateMachine-Role
      PermissionsBoundary: !If [HasBoundary, !Ref PermissionsBoundaryArn, !Ref AWS::NoValue]
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal: { Service: states.amazonaws.com }
            Action: sts:AssumeRole
      Policies:
        - PolicyName: InvokeWorkflowFunctions
          PolicyDocument:
            Version: "2012-10-17"
            Statement:
              - Effect: Allow
                Action: lambda:InvokeFunction
                Resource:
                  - !GetAtt DataCheckingFunction.Arn
                  - !GetAtt FlagApplicationFunction.Arn

  ApplicationProcessingStateMachine:
    Type: AWS::Serverless::StateMachine
    Properties:
      Name: ApplicationProcessingStateMachine
      DefinitionUri: statemachine/account-application-workflow.asl.json
      DefinitionSubstitutions:
        DataCheckingFunctionArn: !GetAtt DataCheckingFunction.Arn
        FlagApplicationFunctionName: !Ref FlagApplicationFunction
      Role: !GetAtt StateMachineRole.Arn

  ApproveApplicationFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: sfn-workshop-ApproveApplication
      CodeUri: functions/account-applications/
      Handler: approve.handler
      Role: !GetAtt FunctionsRole.Arn
      Environment:
        Variables:
          APPLICATIONS_TABLE_NAME: !Ref ApplicationsTable

  DataCheckingFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: sfn-workshop-DataChecking
      CodeUri: functions/data-checking/
      Handler: data-checking.handler
      Role: !GetAtt FunctionsRole.Arn

  FindApplicationsFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: sfn-workshop-FindApplications
      CodeUri: functions/account-applications/
      Handler: find.handler
      Role: !GetAtt FunctionsRole.Arn
      Environment:
        Variables:
          APPLICATIONS_TABLE_NAME: !Ref ApplicationsTable

  FlagApplicationFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: sfn-workshop-FlagApplication
      CodeUri: functions/account-applications/
      Handler: flag.handler
      Role: !GetAtt FunctionsRole.Arn
      Environment:
        Variables:
          APPLICATIONS_TABLE_NAME: !Ref ApplicationsTable

  RejectApplicationFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: sfn-workshop-RejectApplication
      CodeUri: functions/account-applications/
      Handler: reject.handler
      Role: !GetAtt FunctionsRole.Arn
      Environment:
        Variables:
          APPLICATIONS_TABLE_NAME: !Ref ApplicationsTable

  ReviewApplicationFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: sfn-workshop-ReviewApplication
      CodeUri: functions/account-applications/
      Handler: review.handler
      Role: !GetAtt FunctionsRole.Arn
      Environment:
        Variables:
          APPLICATIONS_TABLE_NAME: !Ref ApplicationsTable

  SubmitApplicationFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: sfn-workshop-SubmitApplication
      CodeUri: functions/account-applications/
      Handler: submit.handler
      Role: !GetAtt FunctionsRole.Arn
      Environment:
        Variables:
          APPLICATIONS_TABLE_NAME: !Ref ApplicationsTable
          APPLICATION_PROCESSING_STEP_FUNCTION_ARN: !Ref ApplicationProcessingStateMachine

  ApplicationsTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: !Sub StepFunctionWorkshop-AccountApplications-${AWS::StackName}
      AttributeDefinitions:
        - { AttributeName: id, AttributeType: S }
        - { AttributeName: state, AttributeType: S }
      KeySchema:
        - { AttributeName: id, KeyType: HASH }
      BillingMode: PAY_PER_REQUEST
      GlobalSecondaryIndexes:
        - IndexName: state
          KeySchema:
            - { AttributeName: state, KeyType: HASH }
          Projection: { ProjectionType: ALL }

Outputs:
  DataCheckingFunctionArn:
    Description: Data Checking Function ARN
    Value: !GetAtt DataCheckingFunction.Arn
EOF
```

3. Replace the definition and deploy:

```bash
cat > statemachine/account-application-workflow.asl.json <<'EOF'
{
    "StartAt": "Check Name",
    "States": {
        "Check Name": {
            "Type": "Task",
            "Parameters": {
                "command": "CHECK_NAME",
                "data": {
                    "name.$": "$.application.name"
                }
            },
            "Resource": "${DataCheckingFunctionArn}",
            "ResultPath": "$.checks.name",
            "Next": "Check Address"
        },
        "Check Address": {
            "Type": "Task",
            "Parameters": {
                "command": "CHECK_ADDRESS",
                "data": {
                    "address.$": "$.application.address"
                }
            },
            "Resource": "${DataCheckingFunctionArn}",
            "ResultPath": "$.checks.address",
            "Next": "Review Required?"
        },
        "Review Required?": {
            "Type": "Choice",
            "Choices": [
                {
                    "Variable": "$.checks.name.flagged",
                    "BooleanEquals": true,
                    "Next": "Pending Review"
                },
                {
                    "Variable": "$.checks.address.flagged",
                    "BooleanEquals": true,
                    "Next": "Pending Review"
                }
            ],
            "Default": "Approve Application"
        },
        "Pending Review": {
            "Type": "Task",
            "Resource": "arn:aws:states:::lambda:invoke.waitForTaskToken",
            "Parameters": {
                "FunctionName": "${FlagApplicationFunctionName}",
                "Payload": {
                    "id.$": "$.application.id",
                    "flagType": "REVIEW",
                    "taskToken.$": "$$.Task.Token"
                }
            },
            "ResultPath": "$.review",
            "Next": "Review Approved?"
        },
        "Review Approved?": {
            "Type": "Choice",
            "Choices": [
                {
                    "Variable": "$.review.decision",
                    "StringEquals": "APPROVE",
                    "Next": "Approve Application"
                },
                {
                    "Variable": "$.review.decision",
                    "StringEquals": "REJECT",
                    "Next": "Reject Application"
                }
            ]
        },
        "Reject Application": {
            "Type": "Pass",
            "End": true
        },
        "Approve Application": {
            "Type": "Pass",
            "End": true
        }
    }
}
EOF
sam build && sam deploy
```

   - (1) The new function is added.
   - (2) The state machine is updated.

   ![Deploy](/images/4-improve-workflow/4.2-deploy.png)

#### Test

4. Submit an application that needs a review, list the flagged applications and save the ID:

```bash
cd ~/workshop-dir
aws lambda invoke --function-name sfn-workshop-SubmitApplication --cli-binary-format raw-in-base64-out --payload '{ "name": "Spock", "address": "InvalidAddressFormat" }' out.json && cat out.json; echo
aws lambda invoke --function-name sfn-workshop-FindApplications --cli-binary-format raw-in-base64-out --payload '{ "state": "FLAGGED_FOR_REVIEW" }' out.json >/dev/null && jq '.Items[] | {id, state, taskToken: (.taskToken[0:40] + "...")}' out.json
ID=$(jq -r '.Items[0].id' out.json); echo $ID
```

   - (1) The application is submitted.
   - (2) It is `FLAGGED_FOR_REVIEW` and has a task token (shortened here).

   ![Submit and find](/images/4-improve-workflow/4.2-submit-find.png)

5. In the console, the new execution is still **Running** (1).

   ![Running](/images/4-improve-workflow/4.2-running.png)

6. Open it. **Pending Review** (1) is in progress: the execution is paused and waits for the token.

   ![Waiting](/images/4-improve-workflow/4.2-pending-wait.png)

7. Act as the reviewer and approve the application:

```bash
aws lambda invoke --function-name sfn-workshop-ReviewApplication --cli-binary-format raw-in-base64-out --payload "{ \"id\": \"$ID\", \"decision\": \"APPROVE\" }" out.json >/dev/null && jq '{id, state}' out.json
```

   - (1) The application is `REVIEW_APPROVED`.

   ![Review](/images/4-improve-workflow/4.2-review.png)

8. Refresh the execution and choose **Review Approved?** (1).
   - (2) Its input has `review.decision` = `APPROVE`, the output sent with `SendTaskSuccess`.
   - (3) The execution continued to **Approve Application** and finished.

   ![Approved](/images/4-improve-workflow/4.2-approved.png)
