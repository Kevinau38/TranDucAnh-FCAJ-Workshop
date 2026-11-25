---
title : "Khởi chạy workflow từ service"
weight : 1
chapter : false
pre : " <b> 4.1 </b> "
---

Để đánh dấu đơn ở bước sau, workflow cần ID của đơn. Cách đơn giản nhất là khởi chạy execution ngay trong `SubmitApplication` sau khi lưu đơn, với toàn bộ đơn (gồm cả ID) làm input.

1. Thay `submit.js`. Function giờ gọi `StartExecution` bằng AWS SDK for JavaScript v3. Tên execution là `ApplicationID-<id>`, nên mỗi đơn có đúng một execution. Nếu lời gọi lỗi, đơn vừa lưu sẽ bị xóa.

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

2. Thay `template.yaml`. Có hai thay đổi:
   - `SubmitApplicationFunction` có thêm biến môi trường `APPLICATION_PROCESSING_STEP_FUNCTION_ARN`.
   - `FunctionsRole` có thêm policy `StateMachineAccess` cho phép `states:StartExecution` trên state machine. ARN của state machine được ghép bằng `!Sub` từ tên cố định, tránh tham chiếu vòng giữa role, function và state machine.

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

   - (1) Role có thêm policy mới.
   - (2) Function submit có code và biến môi trường mới.

   ![Deploy](/images/4-improve-workflow/4.1-deploy.png)

#### Kiểm tra

4. Gửi một đơn có địa chỉ không hợp lệ:

```bash
aws lambda invoke --function-name sfn-workshop-SubmitApplication --cli-binary-format raw-in-base64-out --payload '{ "name": "Spock", "address": "AnInvalidAddress" }' out.json && cat out.json; echo
```

   - (1) Đơn được lưu kèm ID.

   ![Gửi đơn](/images/4-improve-workflow/4.1-submit.png)

5. Mở `ApplicationProcessingStateMachine`. Function đã khởi chạy một execution mới tên `ApplicationID-application_...` (1).

   ![Executions](/images/4-improve-workflow/4.1-executions.png)

6. Mở execution và chọn **Pending Review** (1). Đơn trong output đã có `id` (2).

   ![Output có ID](/images/4-improve-workflow/4.1-pending-output.png)
