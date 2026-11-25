---
title : "Cấp quyền bằng AWS SAM"
weight : 3
chapter : false
pre : " <b> 3.3 </b> "
---

Thay vì sửa role bằng tay, bạn chuyển state machine vào SAM template. Definition nằm trong file riêng `statemachine/account-application-workflow.asl.json`. Khi deploy, SAM thay `${DataCheckingFunctionArn}` bằng ARN thật (`DefinitionSubstitutions`), và role mới `FCJ-SFN-StateMachine-Role` chỉ cho phép đúng một việc: gọi function Data Checking.

1. Xóa state machine tạo trên console để không nhầm với state machine mới. Chọn nó trong **State machines**, chọn **Delete**, gõ tên (1) rồi chọn **Delete** (2).

   ![Xóa](/images/3-orchestration/3.3-delete.png)

2. Tạo file definition:

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

3. Thay `template.yaml`. So với bản trước, template thêm `StateMachineRole` và `ApplicationProcessingStateMachine`:

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

   - (1) Changeset thêm state machine và role của nó.

   ![Deploy](/images/3-orchestration/3.3-deploy.png)

#### Kiểm tra state machine mới

5. Trong **State machines**, state machine tạo trên console đã mất và có `ApplicationProcessingStateMachine` (1).

   ![Danh sách](/images/3-orchestration/3.3-sm-list.png)

6. Mở state machine, chọn **Start execution**, nhập input như cũ rồi chạy.
   - (1) Vẫn **Failed**, nhưng vì lý do khác.
   - (2) Role giờ là `FCJ-SFN-StateMachine-Role`.
   - (3) Lỗi `States.Runtime` ở **Check Address**.

   ![Thất bại](/images/3-orchestration/3.3-failed.png)

7. Mở **Cause** (1). **Check Name** đã chạy được, nhưng **Check Address** (2) không tìm thấy `$.application.address`: input của nó chỉ còn `{"flagged":false}`.

   ![Nguyên nhân](/images/3-orchestration/3.3-cause.png)

Vấn đề quyền đã được giải quyết. Lỗi mới đến từ cách state Task tạo output, đó là nội dung của bước tiếp theo.
