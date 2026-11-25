---
title : "Start the workflow from the service"
weight : 1
chapter : false
pre : " <b> 4.1 </b> "
---

To flag an application later, the workflow needs the application ID. The simplest way is to start the execution from `SubmitApplication` right after the application is stored, with the whole application (including its ID) as input.

1. Replace `submit.js`. It now calls `StartExecution` with the AWS SDK for JavaScript v3. The execution name is `ApplicationID-<id>`, so every application has exactly one execution. If the call fails, the stored application is deleted.

```bash
cat > functions/account-applications/submit.js <<'EOF'
const { SFNClient, StartExecutionCommand } = require('@aws-sdk/client-sfn');
const AccountApplications = require('./AccountApplications');

const sfn = new SFNClient({});

exports.handler = async (event) => {
    const { name, address } = event;
    const application = await AccountApplications.create({ name, address, state: 'SUBMITTED' });
    try {
        await sfn.send(new StartExecutionCommand({
            stateMachineArn: process.env.APPLICATION_PROCESSING_STEP_FUNCTION_ARN,
            name: `ApplicationID-${application.id}`,
            input: JSON.stringify({ application })
        }));
        return application;
    } catch (ex) {
        await AccountApplications.delete(application.id);
        throw ex;
    }
};
EOF
```

2. Replace `template.yaml`. Two changes:
   - `SubmitApplicationFunction` gets the environment variable `APPLICATION_PROCESSING_STEP_FUNCTION_ARN`.
   - `FunctionsRole` gets the policy `StateMachineAccess` that allows `states:StartExecution` on the state machine. The state machine ARN is built with `!Sub` from its fixed name, which avoids a circular reference between the role, the functions and the state machine.

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
                Action: [states:StartExecution]
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

3. Deploy:

```bash
sam build && sam deploy
```

   - (1) The role gets the new policy.
   - (2) The submit function gets the new code and variable.

   ![Deploy](/images/4-improve-workflow/4.1-deploy.png)

#### Test

4. Submit an application with an invalid address:

```bash
aws lambda invoke --function-name sfn-workshop-SubmitApplication --cli-binary-format raw-in-base64-out --payload '{ "name": "Spock", "address": "AnInvalidAddress" }' out.json && cat out.json; echo
```

   - (1) The application is stored with its ID.

   ![Submit](/images/4-improve-workflow/4.1-submit.png)

5. Open `ApplicationProcessingStateMachine`. A new execution named `ApplicationID-application_...` (1) was started by the function.

   ![Executions](/images/4-improve-workflow/4.1-executions.png)

6. Open it and choose **Pending Review** (1). The application in the output now has its `id` (2).

   ![Output with ID](/images/4-improve-workflow/4.1-pending-output.png)
