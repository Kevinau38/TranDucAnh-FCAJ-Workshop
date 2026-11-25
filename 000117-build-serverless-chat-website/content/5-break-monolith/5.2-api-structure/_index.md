---
title : "Build the API resources"
weight : 2
chapter : false
pre : " <b> 5.2 </b> "
---

The new API has two resources:

| Resource | Method | Lambda function |
|---|---|---|
| `/conversations` | GET | `Chat-Conversation-GET` |
| `/conversations/{id}` | GET | `Chat-Messages-GET` |
| `/conversations/{id}` | POST | `Chat-Messages-POST` |

These methods use a normal **Lambda** integration (not proxy). A **mapping template** turns the HTTP request into the small event that each function expects, for example the path parameter `id` and the request body.

1. Run the script below in CloudShell. It deletes `/{proxy+}`, creates the two resources and the three methods, adds the mapping templates from `v8/Mapping Templates`, returns the header `Access-Control-Allow-Origin: *` and gives API Gateway permission to invoke each function.

```bash
cat > ~/api-v8.sh <<'EOF'
source ~/chat.env; A=$API_ID; R=ap-northeast-1; ACC=$(aws sts get-caller-identity --query Account --output text)
ROOT=$(aws apigateway get-resources --rest-api-id $A --query "items[?path=='/'].id" --output text)
PROXY=$(aws apigateway get-resources --rest-api-id $A --query "items[?path=='/{proxy+}'].id" --output text)
[ -n "$PROXY" ] && aws apigateway delete-resource --rest-api-id $A --resource-id $PROXY && echo "Deleted /{proxy+}"
CONV=$(aws apigateway create-resource --rest-api-id $A --parent-id $ROOT --path-part conversations --query id --output text)
ID=$(aws apigateway create-resource --rest-api-id $A --parent-id $CONV --path-part '{id}' --query id --output text)
method(){ # resource verb function [template]
  aws apigateway put-method --rest-api-id $A --resource-id $1 --http-method $2 --authorization-type NONE \
    $([ $1 = $ID ] && echo --request-parameters method.request.path.id=true) >/dev/null
  aws apigateway put-integration --rest-api-id $A --resource-id $1 --http-method $2 --type AWS --integration-http-method POST \
    --uri arn:aws:apigateway:$R:lambda:path/2015-03-31/functions/arn:aws:lambda:$R:$ACC:function:$3/invocations \
    --passthrough-behavior WHEN_NO_TEMPLATES ${4:+--request-templates "{\"application/json\":$(jq -Rs . < "$4")}"} >/dev/null
  aws apigateway put-method-response --rest-api-id $A --resource-id $1 --http-method $2 --status-code 200 \
    --response-parameters method.response.header.Access-Control-Allow-Origin=false >/dev/null
  aws apigateway put-integration-response --rest-api-id $A --resource-id $1 --http-method $2 --status-code 200 \
    --response-parameters "{\"method.response.header.Access-Control-Allow-Origin\":\"'*'\"}" >/dev/null
  aws lambda add-permission --function-name $3 --statement-id apigw-$2-$1 --action lambda:InvokeFunction \
    --principal apigateway.amazonaws.com --source-arn "arn:aws:execute-api:$R:$ACC:$A/*/*/*" >/dev/null
  echo "$2 -> $3"
}
T=~/ws2-chatapp/v8/Mapping\ Templates
method $CONV GET Chat-Conversation-GET
method $ID GET Chat-Messages-GET "$T/Chat-Messages-GET-Input.vtl"
method $ID POST Chat-Messages-POST "$T/Chat-Messages-POST-Input.vtl"
EOF
bash ~/api-v8.sh
```

   The `id` path parameter is declared on the `{id}` methods (`method.request.path.id`). The generated SDK in part 5.3 needs it to put the ID in the URL.

2. In the API Gateway console, select `/{id}` and choose **Enable CORS**:
   - (1) Select **Default 4XX** and **Default 5XX**, so error responses also carry the CORS headers.
   - (2) Select **GET** and **POST**.
   - Choose **Save**. Do the same for `/conversations` with **GET**.

   ![Enable CORS](/images/5-break-monolith/5.1-cors-id.png)

3. Select **GET** under `/{id}`, open **Integration request** and scroll to **Mapping templates**.
   - (1) The resource tree now has `/conversations` and `/conversations/{id}`.
   - (2) The template for `application/json` builds `{"id": "..."}` from the path.

   ![Mapping template](/images/5-break-monolith/5.2-mapping-template.png)
