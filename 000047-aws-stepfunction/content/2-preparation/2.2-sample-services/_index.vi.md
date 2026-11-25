---
title : "Tạo các service mẫu"
weight : 2
chapter : false
pre : " <b> 2.2 </b> "
---

Các service mẫu gồm sáu Lambda function và một bảng DynamoDB:

| Function | Handler | Chức năng |
|---|---|---|
| `sfn-workshop-SubmitApplication` | `submit.js` | Lưu đơn mới với trạng thái `SUBMITTED` |
| `sfn-workshop-FlagApplication` | `flag.js` | Đánh dấu đơn `FLAGGED_FOR_REVIEW` hoặc `FLAGGED_WITH_UNPROCESSABLE_DATA` |
| `sfn-workshop-FindApplications` | `find.js` | Liệt kê đơn theo trạng thái |
| `sfn-workshop-ApproveApplication` | `approve.js` | Đánh dấu đơn `APPROVED` |
| `sfn-workshop-RejectApplication` | `reject.js` | Đánh dấu đơn `REJECTED` |
| `sfn-workshop-DataChecking` | `data-checking.js` | Kiểm tra tên hoặc địa chỉ |

Các function dùng AWS SDK for JavaScript v3 có sẵn trong runtime Node.js của Lambda, nên không cần cài thêm gì.

1. Tạo thư mục dự án và code:

```bash
mkdir -p ~/workshop-dir/functions/account-applications ~/workshop-dir/functions/data-checking && cd ~/workshop-dir
cat > functions/account-applications/AccountApplications.js <<'EOF'
const { randomUUID } = require('node:crypto');
const { DynamoDBClient } = require('@aws-sdk/client-dynamodb');
const { DynamoDBDocumentClient, PutCommand, GetCommand, QueryCommand, DeleteCommand } = require('@aws-sdk/lib-dynamodb');

const dynamo = DynamoDBDocumentClient.from(new DynamoDBClient({}));
const TableName = process.env.APPLICATIONS_TABLE_NAME;

module.exports = {
    async create(attributes) {
        const application = { ...attributes, id: `application_${randomUUID()}` };
        await dynamo.send(new PutCommand({ TableName, Item: application }));
        return application;
    },
    async get(id) {
        const result = await dynamo.send(new GetCommand({ TableName, Key: { id } }));
        return result.Item;
    },
    async update(id, attributes) {
        const application = { ...(await this.get(id)), ...attributes };
        await dynamo.send(new PutCommand({ TableName, Item: application }));
        return application;
    },
    async findAllByState({ state, paginationKey }) {
        const result = await dynamo.send(new QueryCommand({
            TableName,
            IndexName: 'state',
            KeyConditionExpression: '#state = :state',
            ExclusiveStartKey: paginationKey || undefined,
            ExpressionAttributeNames: { '#state': 'state' },
            ExpressionAttributeValues: { ':state': state }
        }));
        return { Items: result.Items, Count: result.Count, LastEvaluatedKey: result.LastEvaluatedKey };
    },
    async delete(id) {
        await dynamo.send(new DeleteCommand({ TableName, Key: { id } }));
    }
};
EOF
cat > functions/account-applications/submit.js <<'EOF'
const AccountApplications = require('./AccountApplications');

exports.handler = async (event) => {
    const { name, address } = event;
    return AccountApplications.create({ name, address, state: 'SUBMITTED' });
};
EOF
cat > functions/account-applications/flag.js <<'EOF'
const AccountApplications = require('./AccountApplications');

exports.handler = async (event) => {
    const { id, flagType } = event;
    if (flagType === 'REVIEW') {
        return AccountApplications.update(id, { state: 'FLAGGED_FOR_REVIEW', reason: event.reason });
    }
    if (flagType === 'UNPROCESSABLE_DATA') {
        const reason = JSON.parse(event.errorInfo.Cause).errorMessage;
        return AccountApplications.update(id, { state: 'FLAGGED_WITH_UNPROCESSABLE_DATA', reason });
    }
    throw new Error('flagType must be REVIEW or UNPROCESSABLE_DATA');
};
EOF
cat > functions/account-applications/approve.js <<'EOF'
const AccountApplications = require('./AccountApplications');

exports.handler = async (event) => AccountApplications.update(event.id, { state: 'APPROVED' });
EOF
cat > functions/account-applications/reject.js <<'EOF'
const AccountApplications = require('./AccountApplications');

exports.handler = async (event) => AccountApplications.update(event.id, { state: 'REJECTED' });
EOF
cat > functions/account-applications/find.js <<'EOF'
const AccountApplications = require('./AccountApplications');

exports.handler = async (event) => AccountApplications.findAllByState(event);
EOF
cat > functions/data-checking/data-checking.js <<'EOF'
const checkName = ({ name }) => {
    if (name.toLowerCase().includes('unprocessable_data')) {
        const error = new Error(`Simulated error: Name '${name}' is not possible to check.`);
        error.name = 'UnprocessableDataException';
        throw error;
    }
    return { flagged: name.toLowerCase().includes('evil') };
};

const checkAddress = ({ address }) => ({ flagged: address.match(/(\d+ \w+)|(\w+ \d+)/g) === null });

const commandHandlers = { CHECK_NAME: checkName, CHECK_ADDRESS: checkAddress };

exports.handler = async (event) => commandHandlers[event.command](event.data);
EOF
echo '{ "name": "account-applications", "version": "1.0.0", "private": true }' > functions/account-applications/package.json
echo '{ "name": "data-checking", "version": "1.0.0", "private": true }' > functions/data-checking/package.json
find . -type f | sort
```

   - (1) Thư mục `workshop-dir` có hai service. `AccountApplications.js` chứa các thao tác DynamoDB mà các file khác dùng chung.

   ![File](/images/2-preparation/2.2-files.png)

2. Tạo `template.yaml`. Thay vì để SAM tự tạo mỗi function một role với tên ngẫu nhiên, template khai báo role `FCJ-SFN-Functions-Role` có permissions boundary truyền qua tham số, và mọi function dùng chung role này. Role chỉ được đọc ghi bảng đơn đăng ký.

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

3. Tạo bucket private chứa gói triển khai và file `samconfig.toml` lưu cấu hình deploy. Thay `<your-suffix>` bằng một chuỗi duy nhất, ví dụ tên bạn và ngày. Nếu tài khoản không dùng permissions boundary, đặt `BOUNDARY=` rỗng.

```bash
BUCKET=fcj-sfn-sam-<your-suffix>
BOUNDARY=arn:aws:iam::$(aws sts get-caller-identity --query Account --output text):policy/LabPermissionsBoundary
aws s3 mb s3://$BUCKET --region ap-northeast-1
cat > samconfig.toml <<EOF
version = 0.1
[default.deploy.parameters]
stack_name = "step-functions-workshop"
s3_bucket = "$BUCKET"
s3_prefix = "step-functions-workshop"
region = "ap-northeast-1"
capabilities = "CAPABILITY_NAMED_IAM"
confirm_changeset = false
parameter_overrides = "PermissionsBoundaryArn=$BOUNDARY"
EOF
aws s3 ls | grep fcj-sfn-sam; cat samconfig.toml
```

   - (1) Bucket đã được tạo.
   - (2) `samconfig.toml` lưu tên stack, bucket, Region, `CAPABILITY_NAMED_IAM` (cần vì template đặt tên cho role) và boundary.

   ![samconfig](/images/2-preparation/2.2-samconfig.png)

4. Build và deploy:

```bash
sam build && sam deploy
```

   - (1) CloudFormation tạo bảng, role và sáu function.

   ![Changeset](/images/2-preparation/2.2-changeset.png)

   - (1) Output `DataCheckingFunctionArn` sẽ dùng ở phần sau.
   - (2) Stack `step-functions-workshop` đã được tạo.

   ![Deploy](/images/2-preparation/2.2-deploy.png)
