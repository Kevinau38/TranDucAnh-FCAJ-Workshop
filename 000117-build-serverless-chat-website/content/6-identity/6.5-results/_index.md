---
title : "Check the results"
weight : 5
chapter : false
pre : " <b> 6.5 </b> "
---

1. After signing in, the page **people.html** lists the users of the pool (1). Choose **Start Chat** next to the other user.

   ![People](/images/6-identity/6.5-people.png)

2. Send a message. It appears with your own user name, not `Student` (1).

   ![Chat with Cognito](/images/6-identity/6.5-chat-cognito.png)

3. Choose **Conversations**. The new conversation with the other user is listed (1).

   ![Conversations](/images/6-identity/6.5-conversations.png)

4. Call the API without a token, then look at the message in DynamoDB (replace `kevin` with your user name):

```bash
curl -s -w '\nHTTP %{http_code}\n' $API_URL/conversations; aws dynamodb scan --table-name Chat-Messages --query 'Items[?Sender.S==`kevin`].[Sender.S,Message.S]' --output text
```

   - (1) The request has no `Authorization` header.
   - (2) API Gateway rejects it with `401 Unauthorized`, before any Lambda function runs.
   - (3) The sender of the message is the user name from the ID token.

   ![Unauthorized](/images/6-identity/6.5-unauthorized.png)
