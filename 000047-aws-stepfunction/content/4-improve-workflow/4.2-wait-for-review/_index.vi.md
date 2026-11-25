---
title : "Tạm dừng workflow chờ xem xét"
weight : 2
chapter : false
pre : " <b> 4.2 </b> "
---

Step Functions có ba cách tích hợp với một service:

- **Request response**: gọi service và đi tiếp ngay khi service trả lời. Các task Data Checking làm theo cách này.
- **Run a job** (`.sync`): gọi service và chờ đến khi job xong. Không dùng trong bài.
- **Wait for a callback** (`.waitForTaskToken`): gọi service kèm một **task token**, rồi tạm dừng cho đến khi có ai đó trả token về bằng `SendTaskSuccess` hoặc `SendTaskFailure`.

Bước xem xét dùng cách callback:

- **Pending Review** gọi `FlagApplication` qua `arn:aws:states:::lambda:invoke.waitForTaskToken` và truyền token `$$.Task.Token`. Function lưu token cùng đơn đăng ký.
- Function mới `ReviewApplication` nhận ID và quyết định, cập nhật đơn rồi gọi `SendTaskSuccess` với token đã lưu và `{"decision": ...}`.
- State Choice mới **Review Approved?** đọc `$.review.decision` để sang **Approve Application** hoặc **Reject Application**.

1. Thay `flag.js` (giờ lưu thêm `taskToken`) và tạo `review.js`:

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

2. Thay `template.yaml`. Template thêm `ReviewApplicationFunction`, cho phép `states:SendTaskSuccess` và `states:SendTaskFailure` trong `StateMachineAccess`, cho role của state machine gọi `FlagApplicationFunction`, và truyền `FlagApplicationFunctionName` vào definition:

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

3. Thay definition rồi deploy:

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

   - (1) Function mới được thêm.
   - (2) State machine được cập nhật.

   ![Deploy](/images/4-improve-workflow/4.2-deploy.png)

#### Kiểm tra

4. Gửi một đơn cần xem xét, liệt kê các đơn bị đánh dấu và lưu ID:

```bash
cd ~/workshop-dir
aws lambda invoke --function-name sfn-workshop-SubmitApplication --cli-binary-format raw-in-base64-out --payload '{ "name": "Spock", "address": "InvalidAddressFormat" }' out.json && cat out.json; echo
aws lambda invoke --function-name sfn-workshop-FindApplications --cli-binary-format raw-in-base64-out --payload '{ "state": "FLAGGED_FOR_REVIEW" }' out.json >/dev/null && jq '.Items[] | {id, state, taskToken: (.taskToken[0:40] + "...")}' out.json
ID=$(jq -r '.Items[0].id' out.json); echo $ID
```

   - (1) Đơn đã được gửi.
   - (2) Đơn ở trạng thái `FLAGGED_FOR_REVIEW` và có task token (đã rút gọn).

   ![Gửi và tìm](/images/4-improve-workflow/4.2-submit-find.png)

5. Trên console, execution mới vẫn đang **Running** (1).

   ![Đang chạy](/images/4-improve-workflow/4.2-running.png)

6. Mở execution. **Pending Review** (1) đang thực hiện: execution tạm dừng và chờ token.

   ![Đang chờ](/images/4-improve-workflow/4.2-pending-wait.png)

7. Đóng vai người kiểm duyệt và duyệt đơn:

```bash
aws lambda invoke --function-name sfn-workshop-ReviewApplication --cli-binary-format raw-in-base64-out --payload "{ \"id\": \"$ID\", \"decision\": \"APPROVE\" }" out.json >/dev/null && jq '{id, state}' out.json
```

   - (1) Đơn chuyển sang `REVIEW_APPROVED`.

   ![Xem xét](/images/4-improve-workflow/4.2-review.png)

8. Tải lại execution và chọn **Review Approved?** (1).
   - (2) Input có `review.decision` = `APPROVE`, chính là output gửi kèm `SendTaskSuccess`.
   - (3) Execution chạy tiếp sang **Approve Application** và kết thúc.

   ![Đã duyệt](/images/4-improve-workflow/4.2-approved.png)
