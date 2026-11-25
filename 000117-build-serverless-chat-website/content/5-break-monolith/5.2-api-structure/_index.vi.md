---
title : "Xây dựng resource cho API"
weight : 2
chapter : false
pre : " <b> 5.2 </b> "
---

API mới có hai resource:

| Resource | Method | Lambda function |
|---|---|---|
| `/conversations` | GET | `Chat-Conversation-GET` |
| `/conversations/{id}` | GET | `Chat-Messages-GET` |
| `/conversations/{id}` | POST | `Chat-Messages-POST` |

Các method này dùng integration **Lambda** thường (không phải proxy). Một **mapping template** đổi HTTP request thành event nhỏ mà mỗi function cần, ví dụ path parameter `id` và nội dung body.

1. Chạy script dưới đây trong CloudShell. Script xóa `/{proxy+}`, tạo hai resource và ba method, thêm mapping template trong `v8/Mapping Templates`, trả header `Access-Control-Allow-Origin: *` và cấp quyền cho API Gateway gọi từng function.

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

   Path parameter `id` được khai báo trên các method của `{id}` (`method.request.path.id`). SDK sinh ở phần 5.3 cần khai báo này để đưa ID vào URL.

2. Trong console API Gateway, chọn `/{id}` rồi chọn **Enable CORS**:
   - (1) Chọn **Default 4XX** và **Default 5XX**, để response lỗi cũng có header CORS.
   - (2) Chọn **GET** và **POST**.
   - Chọn **Save**. Làm tương tự với `/conversations`, chọn **GET**.

   ![Enable CORS](/images/5-break-monolith/5.1-cors-id.png)

3. Chọn **GET** dưới `/{id}`, mở **Integration request** rồi cuộn tới **Mapping templates**.
   - (1) Cây resource giờ có `/conversations` và `/conversations/{id}`.
   - (2) Template cho `application/json` tạo `{"id": "..."}` từ path.

   ![Mapping template](/images/5-break-monolith/5.2-mapping-template.png)
