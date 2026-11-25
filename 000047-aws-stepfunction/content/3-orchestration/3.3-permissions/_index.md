---
title : "Grant permissions with AWS SAM"
weight : 3
chapter : false
pre : " <b> 3.3 </b> "
---

Instead of fixing the role by hand, you move the state machine into the SAM template. The definition goes into its own file, `statemachine/account-application-workflow.asl.json`. SAM replaces `${DataCheckingFunctionArn}` with the real ARN when it deploys (`DefinitionSubstitutions`), and a new role `FCJ-SFN-StateMachine-Role` allows exactly one action: invoke the Data Checking function.

1. Delete the console state machine so that it is not confused with the new one. Select it in **State machines**, choose **Delete**, type its name (1) and choose **Delete** (2).

   ![Delete](/images/3-orchestration/3.3-delete.png)

2. Create the definition file:

```bash
cd ~/workshop-dir && mkdir -p statemachine
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
            "Next": "Approve Application"
        },
        "Approve Application": {
            "Type": "Pass",
            "End": true
        }
    }
}
EOF
```

3. Replace `template.yaml`. Compared with the previous version it adds `StateMachineRole` and `ApplicationProcessingStateMachine`:

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

  ApplicationProcessingStateMachine:
    Type: AWS::Serverless::StateMachine
    Properties:
      Name: ApplicationProcessingStateMachine
      DefinitionUri: statemachine/account-application-workflow.asl.json
      DefinitionSubstitutions:
        DataCheckingFunctionArn: !GetAtt DataCheckingFunction.Arn
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

4. Deploy:

```bash
sam build && sam deploy
```

   - (1) The changeset adds the state machine and its role.

   ![Deploy](/images/3-orchestration/3.3-deploy.png)

#### Test the new state machine

5. In **State machines**, the console state machine is gone and `ApplicationProcessingStateMachine` (1) is listed.

   ![State machine list](/images/3-orchestration/3.3-sm-list.png)

6. Open it, choose **Start execution**, enter the same input and start the execution.
   - (1) It still **Failed**, but for another reason.
   - (2) The role is now `FCJ-SFN-StateMachine-Role`.
   - (3) The error is `States.Runtime` in **Check Address**.

   ![Failed](/images/3-orchestration/3.3-failed.png)

7. Expand **Cause** (1). **Check Name** succeeded, but **Check Address** (2) could not find `$.application.address`: its input was only `{"flagged":false}`.

   ![Cause](/images/3-orchestration/3.3-cause.png)

The permission problem is solved. The new error comes from the way a Task state builds its output, which is the topic of the next step.
