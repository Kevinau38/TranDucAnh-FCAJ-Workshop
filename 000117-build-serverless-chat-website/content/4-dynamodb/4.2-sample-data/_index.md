---
title : "Add sample data and permissions"
weight : 2
chapter : false
pre : " <b> 4.2 </b> "
---

1. Add two conversations and one message in each:

   ```bash
   for p in 1:Student 1:Brian 2:Student 2:Frank; do aws dynamodb put-item --table-name Chat-Conversations --item "{\"ConversationId\":{\"S\":\"${p%%:*}\"},\"Username\":{\"S\":\"${p##*:}\"}}" && echo "Chat-Conversations <- $p"; done
   T=$(date +%s%3N)
   aws dynamodb put-item --table-name Chat-Messages --item "{\"ConversationId\":{\"S\":\"1\"},\"Timestamp\":{\"N\":\"$T\"},\"Sender\":{\"S\":\"Brian\"},\"Message\":{\"S\":\"Hello Student, this message comes from DynamoDB.\"}}"
   aws dynamodb put-item --table-name Chat-Messages --item "{\"ConversationId\":{\"S\":\"2\"},\"Timestamp\":{\"N\":\"$T\"},\"Sender\":{\"S\":\"Frank\"},\"Message\":{\"S\":\"Hi, I am Frank.\"}}"
   aws dynamodb scan --table-name Chat-Messages --query 'Items[].[ConversationId.S,Sender.S,Message.S]' --output text
   ```

   - (1) Conversation `1` has `Student` and `Brian`, conversation `2` has `Student` and `Frank`.
   - (2) Each conversation has one message.

   ![Sample data](/images/4-dynamodb/4.2-sample-data.png)

2. Allow the Lambda role to query and write the tables:

   ```bash
   ACC=$(aws sts get-caller-identity --query Account --output text); T=arn:aws:dynamodb:ap-northeast-1:$ACC:table
   aws iam put-role-policy --role-name FCJ-ChatApp-Lambda-Role --policy-name Access-DynamoDB-ChatApp --policy-document "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Action\":[\"dynamodb:BatchWriteItem\",\"dynamodb:PutItem\",\"dynamodb:Query\"],\"Resource\":[\"$T/Chat-Conversations\",\"$T/Chat-Messages\",\"$T/Chat-Conversations/index/Username-ConversationId-index\"]}]}"
   aws iam list-role-policies --role-name FCJ-ChatApp-Lambda-Role --output text
   ```

   - (1) The policy allows only `Query`, `PutItem` and `BatchWriteItem`, on the two tables and the index.
   - (2) The role now has two inline policies.

   ![DynamoDB policy](/images/4-dynamodb/4.2-dynamodb-policy.png)
