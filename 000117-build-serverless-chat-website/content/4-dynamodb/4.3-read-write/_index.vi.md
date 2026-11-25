---
title : "Đọc và ghi tin nhắn"
weight : 3
chapter : false
pre : " <b> 4.3 </b> "
---

Phiên bản `v7` của `Chat-Proxy` đọc cuộc trò chuyện từ DynamoDB và lưu tin nhắn mới khi trình duyệt gửi `POST /conversations/{id}`.

1. Deploy code mới và gọi API:

   ```bash
   cd ~/fn && cp ~/ws2-chatapp/v7/Lambda/Chat-Proxy.mjs index.mjs && zip -q proxy.zip index.mjs
   aws lambda update-function-code --function-name Chat-Proxy --zip-file fileb://proxy.zip --query LastUpdateStatus --output text
   sleep 5; curl -s $API_URL/conversations; echo; curl -s $API_URL/conversations/1; echo
   ```

   - (1) Code đã được thay. Bạn cũng có thể dán `v7/Lambda/Chat-Proxy.mjs` vào console rồi chọn **Deploy**.
   - (2) `GET /conversations` và `GET /conversations/1` giờ trả dữ liệu từ DynamoDB.

   ![Proxy dùng DynamoDB](/images/4-dynamodb/4.3-proxy-dynamodb.png)

2. Upload website `v7`, bản này có thêm chức năng gửi tin nhắn:

   ```bash
   cd ~/ws2-chatapp && aws s3 sync v7/Site s3://$BUCKET --exclude '*desktop.ini' --only-show-errors
   ```

3. Mở `chat.html#1`, nhập tin nhắn rồi chọn **Send**.
   - (1) Tin nhắn mới hiện ở bên phải.

   ![Gửi tin nhắn](/images/4-dynamodb/4.3-send-message.png)

4. Trong console DynamoDB, mở **Explore items** và chọn `Chat-Messages`.
   - (1) Item mới của người gửi `Student` đã nằm trong bảng.

   ![Items](/images/4-dynamodb/4.3-items.png)

Người gửi luôn là `Student` vì code chưa biết người dùng là ai. Phần 6 sẽ sửa bằng Cognito.
