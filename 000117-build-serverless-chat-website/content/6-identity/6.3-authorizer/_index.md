---
title : "Protect the API with a Cognito authorizer"
weight : 3
chapter : false
pre : " <b> 6.3 </b> "
---

1. In the API Gateway console, open **Authorizers** and choose **Create authorizer**.
   - (1) **Authorizer name**: `Cognito`.
   - (2) **Authorizer type**: **Cognito**.
   - (3) **Cognito user pool**: your pool.
   - (4) **Token source**: `Authorization`. The website sends the ID token in this header.
   - (5) Choose **Create authorizer**.

   ![Authorizer](/images/6-identity/6.3-authorizer.png)

2. Run the script below. It adds `/users`, the method `POST /conversations`, sets the authorizer on every method and uses the mapping templates of `v15`. These templates add `cognitoUsername` from `$context.authorizer.claims['cognito:username']` to the event, so a user cannot pretend to be someone else.

```bash
cat > ~/api-v15.sh <<'EOF'
source ~/chat.env; A=$API_ID; R=ap-northeast-1; ACC=$(aws sts get-caller-identity --query Account --output text)
AUTH=$(aws apigateway get-authorizers --rest-api-id $A --query "items[?name=='Cognito'].id" --output text)
res(){ aws apigateway get-resources --rest-api-id $A --query "items[?path=='$1'].id" --output text; }
ROOT=$(res /); CONV=$(res /conversations); ID=$(res '/conversations/{id}')
USERS=$(res /users); [ -z "$USERS" ] && USERS=$(aws apigateway create-resource --rest-api-id $A --parent-id $ROOT --path-part users --query id --output text)
T=~/ws2-chatapp/v15/Mapping\ Templates
method(){ # resource verb function template
  aws apigateway put-method --rest-api-id $A --resource-id $1 --http-method $2 --authorization-type COGNITO_USER_POOLS --authorizer-id $AUTH \
    $([ $1 = $ID ] && echo --request-parameters method.request.path.id=true) >/dev/null 2>&1 || \
  aws apigateway update-method --rest-api-id $A --resource-id $1 --http-method $2 \
    --patch-operations op=replace,path=/authorizationType,value=COGNITO_USER_POOLS op=replace,path=/authorizerId,value=$AUTH >/dev/null
  aws apigateway put-integration --rest-api-id $A --resource-id $1 --http-method $2 --type AWS --integration-http-method POST \
    --uri arn:aws:apigateway:$R:lambda:path/2015-03-31/functions/arn:aws:lambda:$R:$ACC:function:$3/invocations \
    --passthrough-behavior WHEN_NO_TEMPLATES --request-templates "{\"application/json\":$(jq -Rs . < "$T/$4")}" >/dev/null
  aws apigateway put-method-response --rest-api-id $A --resource-id $1 --http-method $2 --status-code 200 \
    --response-parameters method.response.header.Access-Control-Allow-Origin=false >/dev/null 2>&1
  aws apigateway put-integration-response --rest-api-id $A --resource-id $1 --http-method $2 --status-code 200 \
    --response-parameters "{\"method.response.header.Access-Control-Allow-Origin\":\"'*'\"}" >/dev/null
  aws lambda add-permission --function-name $3 --statement-id apigw-$2-$1 --action lambda:InvokeFunction \
    --principal apigateway.amazonaws.com --source-arn "arn:aws:execute-api:$R:$ACC:$A/*/*/*" >/dev/null 2>&1
  echo "$2 $(aws apigateway get-method --rest-api-id $A --resource-id $1 --http-method $2 --query authorizationType --output text) -> $3"
}
method $CONV GET Chat-Conversation-GET Chat-Conversation-GET-Input.vtl
method $CONV POST Chat-Conversation-POST Chat-Conversation-POST-Input.vtl
method $ID GET Chat-Messages-GET Chat-Messages-GET-Input.vtl
method $ID POST Chat-Messages-POST Chat-Messages-POST-Input.vtl
method $USERS GET Chat-Users-GET Chat-Users-GET-Input.vtl
EOF
bash ~/api-v15.sh
```

   - (1) Run the script.
   - (2) All five methods show `COGNITO_USER_POOLS`.

   ![API with identity](/images/6-identity/6.3-api-identity.png)

3. Choose **Enable CORS** on `/users` (select **GET**) and on `/conversations` (select **GET**, **OPTIONS** and **POST**). Select **Default 4XX** and **Default 5XX** too: when the token is missing, the `401` response also needs the CORS headers.
   - (1) Gateway responses.
   - (2) Allowed methods of `/conversations`.

   ![CORS for /conversations](/images/6-identity/6.3-cors-conversations.png)

4. Check the resources:
   - (1) The API now has `/conversations`, `/conversations/{id}` and `/users`.
   - (2) **Authorization** of `GET` and `POST` is **Cognito user pools**. `OPTIONS` stays open for the CORS preflight.

   ![Resources with authorization](/images/6-identity/6.3-resources-auth.png)
