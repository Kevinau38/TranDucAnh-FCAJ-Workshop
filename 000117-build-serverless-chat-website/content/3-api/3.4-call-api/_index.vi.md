---
title : "Deploy API và gọi từ website"
weight : 4
chapter : false
pre : " <b> 3.4 </b> "
---

1. Chọn **Deploy API**:
   - (1) **Stage**: **New stage**, **Stage name**: `prod`.
   - (2) Chọn **Deploy**.

   ![Deploy stage](/images/3-api/3.4-deploy-stage.png)

2. Ở trang **Stages**:
   - (1) Stage `prod` đã được tạo.
   - (2) Copy **Invoke URL**.

   ![Invoke URL](/images/3-api/3.4-invoke-url.png)

3. Lưu thông tin API trong CloudShell, ghi URL vào `v3/Site/js/config.js` rồi upload `v3`:

   ```bash
   echo 'export API_ID=<api-id>' >> ~/chat.env
   echo 'export API_URL=https://<api-id>.execute-api.ap-northeast-1.amazonaws.com/prod' >> ~/chat.env && source ~/chat.env
   cd ~/ws2-chatapp && sed -i "s#^ChatApp.apiEndpoint.*#ChatApp.apiEndpoint = '$API_URL';#" v3/Site/js/config.js
   aws s3 cp v3/Site/js/ s3://$BUCKET/js/ --recursive --exclude '*desktop.ini' --only-show-errors
   ```

   - (1) `config.js` đã trỏ tới API của bạn.
   - (2) `config.js` và `site.js` đã được upload.

   ![Config và upload](/images/3-api/3.4-config-upload.png)

4. Tải lại `chats.html`. Danh sách nhìn vẫn như cũ, nhưng dữ liệu giờ đến từ `GET /conversations` của API chứ không phải file JSON.
