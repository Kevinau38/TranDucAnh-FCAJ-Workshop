---
title : "Complete the workflow"
weight : 3
chapter : false
pre : " <b> 4.3 </b> "
---

**Approve Application** and **Reject Application** are still Pass states. Turn them into Task states that call `ApproveApplicationFunction` and `RejectApplicationFunction` with the application ID, and allow the state machine role to invoke both functions.

1. Replace `template.yaml` (two more functions in `InvokeWorkflowFunctions` and two more `DefinitionSubstitutions`):

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
                  - !GetAtt ApproveApplicationFunction.Arn
                  - !GetAtt RejectApplicationFunction.Arn

  ApplicationProcessingStateMachine:
    Type: AWS::Serverless::StateMachine
    Properties:
      Name: ApplicationProcessingStateMachine
      DefinitionUri: statemachine/account-application-workflow.asl.json
      DefinitionSubstitutions:
        DataCheckingFunctionArn: !GetAtt DataCheckingFunction.Arn
        FlagApplicationFunctionName: !Ref FlagApplicationFunction
        ApproveApplicationFunctionArn: !GetAtt ApproveApplicationFunction.Arn
        RejectApplicationFunctionArn: !GetAtt RejectApplicationFunction.Arn
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

2. Replace the definition and deploy:

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
            "Type": "Task",
            "Parameters": {
                "id.$": "$.application.id"
            },
            "Resource": "${RejectApplicationFunctionArn}",
            "End": true
        },
        "Approve Application": {
            "Type": "Task",
            "Parameters": {
                "id.$": "$.application.id"
            },
            "Resource": "${ApproveApplicationFunctionArn}",
            "End": true
        }
    }
}
EOF
sam build && sam deploy
```

   - (1) The state machine role and the state machine are updated.
   - (2) The deployment is complete.

   ![Deploy](/images/4-improve-workflow/4.3-deploy.png)

The basic workflow is complete. In the next part you learn what to do when something goes wrong during an execution.
