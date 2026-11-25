---
title : "Sinh JavaScript SDK"
weight : 3
chapter : false
pre : " <b> 5.3 </b> "
---

API Gateway sinh được JavaScript client cho một stage đã deploy. Phiên bản `v9` của website dùng client này thay vì tự ghép URL.

1. Deploy API, sinh SDK rồi upload cùng website `v9`:

   ```bash
   cd ~ && aws apigateway create-deployment --rest-api-id $API_ID --stage-name prod --query id --output text
   aws apigateway get-sdk --rest-api-id $API_ID --stage-name prod --sdk-type javascript sdk.zip >/dev/null
   rm -rf apiGateway-js-sdk && unzip -q sdk.zip && ls apiGateway-js-sdk
   aws s3 sync apiGateway-js-sdk s3://$BUCKET/js/apiGateway-js-sdk --only-show-errors
   aws s3 sync ws2-chatapp/v9/Site s3://$BUCKET --exclude '*desktop.ini' --only-show-errors && echo SDK-AND-V9-UPLOADED
   ```

   - (1) Deployment được tạo và SDK được tải về. Trên console tương ứng với **Stages** > **Stage actions** > **Generate SDK**.
   - (2) Thư mục SDK (`apigClient.js` và `lib`) và website đã được upload.

   ![Upload SDK](/images/5-break-monolith/5.3-sdk-upload.png)

2. Mở `chat.html#2` và gửi một tin nhắn.
   - (1) Tin nhắn được lưu qua `apigClient.conversationsIdPost`.

   ![Website dùng SDK](/images/5-break-monolith/5.3-site-sdk.png)

{{% notice tip %}}
Mỗi lần thay đổi API (resource, method, tham số), bạn phải deploy lại stage và sinh SDK mới. SDK cũ không biết các method mới.
{{% /notice %}}
