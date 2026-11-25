---
title : "Create the sample services"
weight : 2
chapter : false
pre : " <b> 2.2 </b> "
---

The sample services are six Lambda functions and one DynamoDB table:

| Function | Handler | What it does |
|---|---|---|
| `sfn-workshop-SubmitApplication` | `submit.js` | Stores a new application with state `SUBMITTED` |
| `sfn-workshop-FlagApplication` | `flag.js` | Marks an application `FLAGGED_FOR_REVIEW` or `FLAGGED_WITH_UNPROCESSABLE_DATA` |
| `sfn-workshop-FindApplications` | `find.js` | Lists applications in a given state |
| `sfn-workshop-ApproveApplication` | `approve.js` | Marks an application `APPROVED` |
| `sfn-workshop-RejectApplication` | `reject.js` | Marks an application `REJECTED` |
| `sfn-workshop-DataChecking` | `data-checking.js` | Checks a name or an address |

The functions use the AWS SDK for JavaScript v3 that is included in the Lambda Node.js runtime, so there is nothing to install.

1. Create the project folder and the code:

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

   - (1) The folder `workshop-dir` has the two services. `AccountApplications.js` holds the DynamoDB operations used by the other files.

   ![Files](/images/2-preparation/2.2-files.png)

2. Create `template.yaml`. Instead of letting SAM create one role per function with a random name, the template declares the role `FCJ-SFN-Functions-Role` with the permissions boundary given as a parameter, and every function uses it. The role can only read and write the applications table.

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

3. Create a private bucket for the deployment packages and the file `samconfig.toml` with the deployment settings. Replace `<your-suffix>` with something unique, such as your name and the date. If your account has no permissions boundary, set `BOUNDARY=` to an empty value.

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

   - (1) The bucket is created.
   - (2) `samconfig.toml` keeps the stack name, the bucket, the Region, `CAPABILITY_NAMED_IAM` (needed because the template gives the roles a name) and the boundary.

   ![samconfig](/images/2-preparation/2.2-samconfig.png)

4. Build and deploy:

```bash
sam build && sam deploy
```

   - (1) CloudFormation creates the table, the role and the six functions.

   ![Changeset](/images/2-preparation/2.2-changeset.png)

   - (1) The output `DataCheckingFunctionArn` is used later.
   - (2) The stack `step-functions-workshop` is created.

   ![Deploy](/images/2-preparation/2.2-deploy.png)
