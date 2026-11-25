---
title : "Create the tables"
weight : 1
chapter : false
pre : " <b> 4.1 </b> "
---

1. Open the **DynamoDB** console and choose **Create table**.
   - (1) **Table name**: `Chat-Messages`.
   - (2) **Partition key**: `ConversationId` (String).
   - (3) **Sort key**: `Timestamp` (**Number**). Messages of one conversation are sorted by time.
   - Keep **Default settings** (on-demand capacity) and choose **Create table**.

   ![Chat-Messages](/images/4-dynamodb/4.1-table-messages.png)

2. The second table needs a global secondary index to find the conversations of a user. Create it in CloudShell:

   ```bash
   aws dynamodb create-table --table-name Chat-Conversations \
     --attribute-definitions AttributeName=ConversationId,AttributeType=S AttributeName=Username,AttributeType=S \
     --key-schema AttributeName=ConversationId,KeyType=HASH AttributeName=Username,KeyType=RANGE \
     --billing-mode PAY_PER_REQUEST \
     --global-secondary-indexes 'IndexName=Username-ConversationId-index,KeySchema=[{AttributeName=Username,KeyType=HASH},{AttributeName=ConversationId,KeyType=RANGE}],Projection={ProjectionType=ALL}' \
     --query 'TableDescription.[TableName,TableStatus,GlobalSecondaryIndexes[0].IndexName]' --output text
   ```

   - (1) The command creates `Chat-Conversations` with key `ConversationId` + `Username`.
   - (2) The index `Username-ConversationId-index` uses the opposite key order. The Lambda code queries it by name.

   ![Chat-Conversations](/images/4-dynamodb/4.1-table-conversations.png)
