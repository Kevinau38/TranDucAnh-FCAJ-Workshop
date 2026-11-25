---
title : "Read and write messages"
weight : 3
chapter : false
pre : " <b> 4.3 </b> "
---

Version `v7` of `Chat-Proxy` reads conversations from DynamoDB and saves a new message when the browser sends `POST /conversations/{id}`.

1. Deploy the new code and call the API:

   ```bash
   cd ~/fn && cp ~/ws2-chatapp/v7/Lambda/Chat-Proxy.mjs index.mjs && zip -q proxy.zip index.mjs
   aws lambda update-function-code --function-name Chat-Proxy --zip-file fileb://proxy.zip --query LastUpdateStatus --output text
   sleep 5; curl -s $API_URL/conversations; echo; curl -s $API_URL/conversations/1; echo
   ```

   - (1) The code is replaced. You can also paste `v7/Lambda/Chat-Proxy.mjs` in the console and choose **Deploy**.
   - (2) `GET /conversations` and `GET /conversations/1` now return data from DynamoDB.

   ![Proxy with DynamoDB](/images/4-dynamodb/4.3-proxy-dynamodb.png)

2. Upload the `v7` site, which adds the send feature:

   ```bash
   cd ~/ws2-chatapp && aws s3 sync v7/Site s3://$BUCKET --exclude '*desktop.ini' --only-show-errors
   ```

3. Open `chat.html#1`, type a message and choose **Send**.
   - (1) The new message appears on the right side.

   ![Send a message](/images/4-dynamodb/4.3-send-message.png)

4. In the DynamoDB console, open **Explore items** and select `Chat-Messages`.
   - (1) The new item from sender `Student` is in the table.

   ![Items](/images/4-dynamodb/4.3-items.png)

The sender is always `Student` because the code does not know who the user is yet. You fix this with Cognito in part 6.
