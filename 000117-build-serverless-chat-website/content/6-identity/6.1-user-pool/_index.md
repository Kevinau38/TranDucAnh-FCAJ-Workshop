---
title : "Create the user pool"
weight : 1
chapter : false
pre : " <b> 6.1 </b> "
---

1. Open the **Amazon Cognito** console and choose **Create user pool** (the page **Set up resources for your application**).
   - (1) **Application type**: **Single-page application (SPA)**. The app client has no secret, which is what a browser application needs.
   - (2) **Name your application**: `chatapp`.

   ![Application type](/images/6-identity/6.1-pool-app.png)

2. Under **Configure options**:
   - (1) **Options for sign-in identifiers**: **Username**.
   - (2) Keep **Enable self-registration** selected, and add `email` to **Required attributes for sign-up**. Cognito sends the confirmation code to this email.
   - Leave **Return URL** empty and choose **Create user directory**.

   ![Sign-in options](/images/6-identity/6.1-pool-options.png)

3. Cognito names the pool `User pool - <random>`. Save the pool ID and the app client ID in CloudShell:

   ```bash
   POOL_ID=$(aws cognito-idp list-user-pools --max-results 60 --query "UserPools[?Name=='User pool - <random>'].Id" --output text)
   CLIENT_ID=$(aws cognito-idp list-user-pool-clients --user-pool-id $POOL_ID --query 'UserPoolClients[0].ClientId' --output text)
   echo "export POOL_ID=$POOL_ID CLIENT_ID=$CLIENT_ID" >> ~/chat.env
   aws cognito-idp describe-user-pool-client --user-pool-id $POOL_ID --client-id $CLIENT_ID --query 'UserPoolClient.[ClientName,ExplicitAuthFlows]' --output text
   ```

   The app client `chatapp` allows `ALLOW_USER_SRP_AUTH`, the flow that the Cognito JavaScript library uses to sign in.
