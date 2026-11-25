---
title : "Tạo Lambda function đầu tiên"
weight : 2
chapter : false
pre : " <b> 3.2 </b> "
---

1. Mở console **Lambda**, chọn **Create function**.
   - (1) **Author from scratch**.
   - (2) **Function name**: `Chat-Proxy`.
   - (3) **Runtime**: giữ runtime **Node.js** mới nhất (lúc viết bài là Node.js 26.x).

   ![Tạo function](/images/3-api/3.2-create-function.png)

2. Mở **Additional settings**:
   - (1) Bật **Custom execution role**.
   - (2) Chọn role có sẵn `FCJ-ChatApp-Lambda-Role`.
   - (3) Chọn **Save**, rồi **Create Event Function**.

   ![Execution role](/images/3-api/3.2-role.png)

3. Ở tab **Code**, thay nội dung `index.mjs` bằng code trong `v3/Lambda/Chat-Proxy.mjs`. Đổi `<your bucket name>` thành tên bucket của bạn rồi chọn **Deploy**. Bạn cũng có thể deploy từ CloudShell:

   ```bash
   mkdir -p ~/fn && cd ~/fn
   sed "s/<your bucket name>/$BUCKET/" ~/ws2-chatapp/v3/Lambda/Chat-Proxy.mjs > index.mjs && zip -q proxy.zip index.mjs
   aws lambda update-function-code --function-name Chat-Proxy --zip-file fileb://proxy.zip --query LastUpdateStatus --output text
   ```

   - (1) Function đọc `data/conversations.json` trong bucket của bạn.
   - (2) **Deploy** lưu code.

   ![Code của function](/images/3-api/3.2-code.png)

4. Mở tab **Test**, tạo event tên `Get-Data-From-S3` với JSON `{}`, chọn **Save** rồi **Test**.
   - (1) Kết quả có `statusCode` `200`, `body` chứa hai cuộc trò chuyện đọc từ S3.

   ![Kết quả test](/images/3-api/3.2-test-result.png)
