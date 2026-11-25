---
title : "Tạo bảng"
weight : 1
chapter : false
pre : " <b> 4.1 </b> "
---

1. Mở console **DynamoDB**, chọn **Create table**.
   - (1) **Table name**: `Chat-Messages`.
   - (2) **Partition key**: `ConversationId` (String).
   - (3) **Sort key**: `Timestamp` (**Number**). Tin nhắn trong một cuộc trò chuyện được sắp theo thời gian.
   - Giữ **Default settings** (on-demand) rồi chọn **Create table**.

   ![Chat-Messages](/images/4-dynamodb/4.1-table-messages.png)

2. Bảng thứ hai cần một global secondary index để tìm các cuộc trò chuyện của một người. Tạo bảng này trong CloudShell:

   ```bash
   aws dynamodb create-table --table-name Chat-Conversations \
     --attribute-definitions AttributeName=ConversationId,AttributeType=S AttributeName=Username,AttributeType=S \
     --key-schema AttributeName=ConversationId,KeyType=HASH AttributeName=Username,KeyType=RANGE \
     --billing-mode PAY_PER_REQUEST \
     --global-secondary-indexes 'IndexName=Username-ConversationId-index,KeySchema=[{AttributeName=Username,KeyType=HASH},{AttributeName=ConversationId,KeyType=RANGE}],Projection={ProjectionType=ALL}' \
     --query 'TableDescription.[TableName,TableStatus,GlobalSecondaryIndexes[0].IndexName]' --output text
   ```

   - (1) Lệnh tạo bảng `Chat-Conversations` với khóa `ConversationId` + `Username`.
   - (2) Index `Username-ConversationId-index` đảo thứ tự khóa. Code Lambda gọi index theo đúng tên này.

   ![Chat-Conversations](/images/4-dynamodb/4.1-table-conversations.png)
