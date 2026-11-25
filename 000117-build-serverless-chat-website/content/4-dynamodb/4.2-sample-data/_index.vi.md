---
title : "Thêm dữ liệu mẫu và quyền truy cập"
weight : 2
chapter : false
pre : " <b> 4.2 </b> "
---

1. Thêm hai cuộc trò chuyện, mỗi cuộc một tin nhắn:

   ```bash
   for p in 1:Student 1:Brian 2:Student 2:Frank; do aws dynamodb put-item --table-name Chat-Conversations --item "{\"ConversationId\":{\"S\":\"${p%%:*}\"},\"Username\":{\"S\":\"${p##*:}\"}}" && echo "Chat-Conversations <- $p"; done
   T=$(date +%s%3N)
   aws dynamodb put-item --table-name Chat-Messages --item "{\"ConversationId\":{\"S\":\"1\"},\"Timestamp\":{\"N\":\"$T\"},\"Sender\":{\"S\":\"Brian\"},\"Message\":{\"S\":\"Hello Student, this message comes from DynamoDB.\"}}"
   aws dynamodb put-item --table-name Chat-Messages --item "{\"ConversationId\":{\"S\":\"2\"},\"Timestamp\":{\"N\":\"$T\"},\"Sender\":{\"S\":\"Frank\"},\"Message\":{\"S\":\"Hi, I am Frank.\"}}"
   aws dynamodb scan --table-name Chat-Messages --query 'Items[].[ConversationId.S,Sender.S,Message.S]' --output text
   ```

   - (1) Cuộc trò chuyện `1` gồm `Student` và `Brian`, cuộc `2` gồm `Student` và `Frank`.
   - (2) Mỗi cuộc trò chuyện có một tin nhắn.

   ![Dữ liệu mẫu](/images/4-dynamodb/4.2-sample-data.png)

2. Cho role của Lambda quyền truy vấn và ghi vào bảng:

   ```bash
   ACC=$(aws sts get-caller-identity --query Account --output text); T=arn:aws:dynamodb:ap-northeast-1:$ACC:table
   aws iam put-role-policy --role-name FCJ-ChatApp-Lambda-Role --policy-name Access-DynamoDB-ChatApp --policy-document "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Action\":[\"dynamodb:BatchWriteItem\",\"dynamodb:PutItem\",\"dynamodb:Query\"],\"Resource\":[\"$T/Chat-Conversations\",\"$T/Chat-Messages\",\"$T/Chat-Conversations/index/Username-ConversationId-index\"]}]}"
   aws iam list-role-policies --role-name FCJ-ChatApp-Lambda-Role --output text
   ```

   - (1) Policy chỉ cho `Query`, `PutItem`, `BatchWriteItem` trên hai bảng và index.
   - (2) Role giờ có hai inline policy.

   ![Policy DynamoDB](/images/4-dynamodb/4.2-dynamodb-policy.png)
