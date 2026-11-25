---
title : "Deploy the final website and sign up"
weight : 4
chapter : false
pre : " <b> 6.4 </b> "
---

The final website is made of the pages of `v14` (sign in, people, conversations, chat), `signup.html` of `v12`, `confirm.html` of `v13` and `js/site.js` of `v15`. It uses the Amazon Cognito Identity library to sign in and sends the ID token to the API.

1. Build the site folder. `config.js` holds the IDs of your pool and app client, and the Cognito library is taken from the npm package `amazon-cognito-identity-js`:

```bash
source ~/chat.env; cd ~
rm -rf site && cp -r ws2-chatapp/v1/Site site && for v in v12 v13 v14 v15; do cp -r ws2-chatapp/$v/Site/. site/; done
find site -name desktop.ini -delete
printf "var poolData = {\n    UserPoolId: '%s',\n    ClientId: '%s'\n};" $POOL_ID $CLIENT_ID > site/js/config.js
(cd /tmp && npm pack amazon-cognito-identity-js --silent >/dev/null && tar xzf amazon-cognito-identity-js-*.tgz && cp package/dist/amazon-cognito-identity.min.js ~/site/js/)
```

2. Deploy the API again, download the new SDK and upload everything:

```bash
aws apigateway create-deployment --rest-api-id $API_ID --stage-name prod --query id --output text && aws apigateway get-sdk --rest-api-id $API_ID --stage-name prod --sdk-type javascript sdk.zip >/dev/null && rm -rf apiGateway-js-sdk && unzip -q sdk.zip && grep -o 'apigClient\.[a-zA-Z]*' apiGateway-js-sdk/apigClient.js | sort -u | tr '\n' ' '; echo; aws s3 sync apiGateway-js-sdk s3://$BUCKET/js/apiGateway-js-sdk --only-show-errors && aws s3 sync site s3://$BUCKET --only-show-errors && aws s3 ls s3://$BUCKET/ && aws s3 ls s3://$BUCKET/js/ | grep -v PRE
```

   - (1) Deploy the stage `prod` and generate the SDK.
   - (2) The SDK now has the functions `conversationsPost`, `usersGet` and the others.
   - (3) The new pages and `amazon-cognito-identity.min.js` are in the bucket.

   ![Final upload](/images/6-identity/6.4-final-upload.png)

3. After confirming the email, `confirm.html` sends the user to `/`. CloudFront must know which object to return for this path, otherwise S3 answers `AccessDenied`. In the CloudFront console open your distribution, choose **Edit** in **Settings** and enter **Default root object** `index.html` (1). Choose **Save changes** and wait until **Last modified** shows a date.

   ![Default root object](/images/6-identity/6.4-default-root.png)

4. Open `https://<CloudFront domain>/signup.html`.
   - (1) Enter a user name, an email you can read and a password of your choice (at least 8 characters with upper case, lower case, number and symbol).
   - (2) Choose **Sign Up**.

   ![Sign up](/images/6-identity/6.4-signup.png)

5. Cognito emails a verification code. On the page `confirm.html`, enter the user name and the code, then sign in on the home page with your user name and password.

{{% notice tip %}}
To have someone to chat with, create a second user from CloudShell. The user only needs to exist in the pool, it does not have to sign in:

```bash
aws cognito-idp admin-create-user --user-pool-id $POOL_ID --username brian --user-attributes Name=email,Value=brian@example.com --message-action SUPPRESS --query User.Username --output text
```
{{% /notice %}}
