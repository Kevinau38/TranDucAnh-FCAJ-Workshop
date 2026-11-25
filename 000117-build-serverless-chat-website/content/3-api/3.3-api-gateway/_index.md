---
title : "Add API Gateway in front of Lambda"
weight : 3
chapter : false
pre : " <b> 3.3 </b> "
---

1. Open the **API Gateway** console, choose **Create an API**, then **Build** under **REST API**.
   - (1) **New API**.
   - (2) **API name**: `Chat-API`.
   - (3) **Endpoint type**: **Regional**. The site is already served by CloudFront, so the API does not need the edge-optimized type.
   - Choose **Create API**.

   ![Create API](/images/3-api/3.3-create-api.png)

2. Choose **Create resource**:
   - (1) Turn on **Proxy resource**.
   - (2) **Resource name**: `{proxy+}`.
   - (3) Select **CORS (Cross Origin Resource Sharing)**. API Gateway adds an `OPTIONS` method so the browser may call the API from the CloudFront domain.
   - (4) Choose **Create resource**.

   ![Proxy resource](/images/3-api/3.3-proxy-resource.png)

3. Select the `ANY` method under `/{proxy+}` and choose **Edit integration**:
   - (1) **Integration type**: **Lambda function**, and turn on **Lambda proxy integration**. API Gateway passes the whole request to Lambda and returns the `statusCode`, `headers` and `body` that the function builds.
   - (2) **Lambda function**: `Chat-Proxy`.
   - Choose **Save**. API Gateway adds the permission to invoke the function.

   ![Integration](/images/3-api/3.3-integration.png)

4. Open the **Test** tab of `ANY`, choose method **GET**, enter `conversations` in **proxy** and choose **Test**.
   - (1) **Status** is `200`.
   - (2) The **Response body** is the content of `conversations.json`.

   ![Test the API](/images/3-api/3.3-test-api.png)
