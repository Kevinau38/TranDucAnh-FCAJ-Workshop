---
title : "Create the user functions"
weight : 2
chapter : false
pre : " <b> 6.2 </b> "
---

Two new functions are needed: `Chat-Users-GET` lists the users of the pool, and `Chat-Conversation-POST` starts a new conversation. The existing functions must stop using the fixed name `Student`.

1. Create a separate role for `Chat-Users-GET`. It can only call `ListUsers` on your pool:

   ```bash
   source ~/chat.env; cd ~/fn
   aws iam create-role --role-name FCJ-ChatApp-Cognito-Role --assume-role-policy-document file://$HOME/ws2-chatapp/lambda-trust.json ${BOUNDARY:+--permissions-boundary $BOUNDARY} --query Role.RoleName --output text
   aws iam attach-role-policy --role-name FCJ-ChatApp-Cognito-Role --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
   aws iam put-role-policy --role-name FCJ-ChatApp-Cognito-Role --policy-name Lambda-User-Pool-ChatApp --policy-document "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Action\":\"cognito-idp:ListUsers\",\"Resource\":\"arn:aws:cognito-idp:ap-northeast-1:$(aws sts get-caller-identity --query Account --output text):userpool/$POOL_ID\"}]}" && echo COGNITO-ROLE-OK
   ```

   - (1) The role, the managed log policy and the inline `ListUsers` policy are created.
   - (2) The role `FCJ-ChatApp-Cognito-Role` is ready.

   ![Cognito role](/images/6-identity/6.2-cognito-role.png)

2. Create the two functions and update the two existing ones:

   ```bash
   CR=$(aws iam get-role --role-name FCJ-ChatApp-Cognito-Role --query Role.Arn --output text)
   sed "s/<user pool id>/$POOL_ID/" ~/ws2-chatapp/v10/Lambda/Chat-Users-GET.mjs > index.mjs && zip -q users.zip index.mjs
   aws lambda create-function --function-name Chat-Users-GET --runtime $RT --handler index.handler --role $CR --zip-file fileb://users.zip --query '[FunctionName,State]' --output text
   cp ~/ws2-chatapp/v11/Lambda/Chat-Conversation-POST.mjs index.mjs && zip -q cpost.zip index.mjs
   aws lambda create-function --function-name Chat-Conversation-POST --runtime $RT --handler index.handler --role $ROLE_ARN --zip-file fileb://cpost.zip --query '[FunctionName,State]' --output text
   for f in Chat-Conversation-GET Chat-Messages-POST; do
     sed "s/{S: 'Student'}/{S: event.cognitoUsername}/" ~/ws2-chatapp/v8/Lambda/$f.mjs > index.mjs && grep -c cognitoUsername index.mjs && zip -q $f.zip index.mjs
     aws lambda update-function-code --function-name $f --zip-file fileb://$f.zip --query FunctionName --output text
   done
   ```

   - (1) `Chat-Users-GET` uses the new role, `Chat-Conversation-POST` uses `FCJ-ChatApp-Lambda-Role`.
   - (2) Both functions are created.
   - (3) `Chat-Conversation-GET` and `Chat-Messages-POST` now read the user name from `event.cognitoUsername`.

   ![New functions](/images/6-identity/6.2-new-functions.png)
