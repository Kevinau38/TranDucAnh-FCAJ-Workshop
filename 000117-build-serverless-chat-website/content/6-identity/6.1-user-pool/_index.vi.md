---
title : "Tạo user pool"
weight : 1
chapter : false
pre : " <b> 6.1 </b> "
---

1. Mở console **Amazon Cognito**, chọn **Create user pool** (trang **Set up resources for your application**).
   - (1) **Application type**: **Single-page application (SPA)**. App client không có secret, đúng với ứng dụng chạy trên trình duyệt.
   - (2) **Name your application**: `chatapp`.

   ![Loại ứng dụng](/images/6-identity/6.1-pool-app.png)

2. Ở **Configure options**:
   - (1) **Options for sign-in identifiers**: **Username**.
   - (2) Giữ chọn **Enable self-registration**, thêm `email` vào **Required attributes for sign-up**. Cognito gửi mã xác nhận về email này.
   - Để trống **Return URL** rồi chọn **Create user directory**.

   ![Tùy chọn đăng nhập](/images/6-identity/6.1-pool-options.png)

3. Cognito đặt tên pool là `User pool - <ngẫu nhiên>`. Lưu ID của pool và app client trong CloudShell:

   ```bash
   POOL_ID=$(aws cognito-idp list-user-pools --max-results 60 --query "UserPools[?Name=='User pool - <ngẫu nhiên>'].Id" --output text)
   CLIENT_ID=$(aws cognito-idp list-user-pool-clients --user-pool-id $POOL_ID --query 'UserPoolClients[0].ClientId' --output text)
   echo "export POOL_ID=$POOL_ID CLIENT_ID=$CLIENT_ID" >> ~/chat.env
   aws cognito-idp describe-user-pool-client --user-pool-id $POOL_ID --client-id $CLIENT_ID --query 'UserPoolClient.[ClientName,ExplicitAuthFlows]' --output text
   ```

   App client `chatapp` cho phép `ALLOW_USER_SRP_AUTH`, luồng đăng nhập mà thư viện JavaScript của Cognito sử dụng.
