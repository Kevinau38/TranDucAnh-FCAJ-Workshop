---
title : "Create the first Lambda function"
weight : 2
chapter : false
pre : " <b> 3.2 </b> "
---

1. Open the **Lambda** console and choose **Create function**.
   - (1) **Author from scratch**.
   - (2) **Function name**: `Chat-Proxy`.
   - (3) **Runtime**: keep the latest **Node.js** runtime (Node.js 26.x at the time of writing).

   ![Create function](/images/3-api/3.2-create-function.png)

2. Expand **Additional settings**:
   - (1) Turn on **Custom execution role**.
   - (2) Choose the existing role `FCJ-ChatApp-Lambda-Role`.
   - (3) Choose **Save**, then **Create Event Function**.

   ![Execution role](/images/3-api/3.2-role.png)

3. On the **Code** tab, replace the content of `index.mjs` with the code of `v3/Lambda/Chat-Proxy.mjs`. Change `<your bucket name>` to your bucket and choose **Deploy**. You can also deploy it from CloudShell:

   ```bash
   mkdir -p ~/fn && cd ~/fn
   sed "s/<your bucket name>/$BUCKET/" ~/ws2-chatapp/v3/Lambda/Chat-Proxy.mjs > index.mjs && zip -q proxy.zip index.mjs
   aws lambda update-function-code --function-name Chat-Proxy --zip-file fileb://proxy.zip --query LastUpdateStatus --output text
   ```

   - (1) The function reads `data/conversations.json` from your bucket.
   - (2) **Deploy** saves the code.

   ![Function code](/images/3-api/3.2-code.png)

4. Open the **Test** tab, create an event named `Get-Data-From-S3` with the JSON `{}`, choose **Save**, then **Test**.
   - (1) The response has `statusCode` `200`, and the `body` contains the two conversations from S3.

   ![Test result](/images/3-api/3.2-test-result.png)
