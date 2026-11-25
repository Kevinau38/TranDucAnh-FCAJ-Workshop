---
title : "Generate the JavaScript SDK"
weight : 3
chapter : false
pre : " <b> 5.3 </b> "
---

API Gateway can generate a JavaScript client for a deployed stage. Version `v9` of the site uses this client instead of building URLs by hand.

1. Deploy the API, generate the SDK and upload it with the `v9` site:

   ```bash
   cd ~ && aws apigateway create-deployment --rest-api-id $API_ID --stage-name prod --query id --output text
   aws apigateway get-sdk --rest-api-id $API_ID --stage-name prod --sdk-type javascript sdk.zip >/dev/null
   rm -rf apiGateway-js-sdk && unzip -q sdk.zip && ls apiGateway-js-sdk
   aws s3 sync apiGateway-js-sdk s3://$BUCKET/js/apiGateway-js-sdk --only-show-errors
   aws s3 sync ws2-chatapp/v9/Site s3://$BUCKET --exclude '*desktop.ini' --only-show-errors && echo SDK-AND-V9-UPLOADED
   ```

   - (1) The deployment is created and the SDK is downloaded. The console offers the same: **Stages** > **Stage actions** > **Generate SDK**.
   - (2) The SDK folder (`apigClient.js` and `lib`) and the site are uploaded.

   ![SDK upload](/images/5-break-monolith/5.3-sdk-upload.png)

2. Open `chat.html#2` and send a message.
   - (1) The message is saved through `apigClient.conversationsIdPost`.

   ![Site with SDK](/images/5-break-monolith/5.3-site-sdk.png)

{{% notice tip %}}
Every time you change the API (resources, methods, parameters), deploy the stage again and generate a new SDK. An old SDK does not know the new methods.
{{% /notice %}}
