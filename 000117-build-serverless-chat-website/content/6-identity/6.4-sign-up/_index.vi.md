---
title : "Triển khai website hoàn chỉnh và đăng ký"
weight : 4
chapter : false
pre : " <b> 6.4 </b> "
---

Website hoàn chỉnh gồm các trang của `v14` (đăng nhập, danh sách người dùng, danh sách cuộc trò chuyện, chat), `signup.html` của `v12`, `confirm.html` của `v13` và `js/site.js` của `v15`. Website dùng thư viện Amazon Cognito Identity để đăng nhập và gửi ID token tới API.

1. Tạo thư mục website. `config.js` chứa ID của pool và app client, thư viện Cognito lấy từ gói npm `amazon-cognito-identity-js`:

```bash
source ~/chat.env; cd ~
rm -rf site && cp -r ws2-chatapp/v1/Site site && for v in v12 v13 v14 v15; do cp -r ws2-chatapp/$v/Site/. site/; done
find site -name desktop.ini -delete
printf "var poolData = {\n    UserPoolId: '%s',\n    ClientId: '%s'\n};" $POOL_ID $CLIENT_ID > site/js/config.js
(cd /tmp && npm pack amazon-cognito-identity-js --silent >/dev/null && tar xzf amazon-cognito-identity-js-*.tgz && cp package/dist/amazon-cognito-identity.min.js ~/site/js/)
```

2. Deploy lại API, tải SDK mới và upload toàn bộ:

```bash
aws apigateway create-deployment --rest-api-id $API_ID --stage-name prod --query id --output text && aws apigateway get-sdk --rest-api-id $API_ID --stage-name prod --sdk-type javascript sdk.zip >/dev/null && rm -rf apiGateway-js-sdk && unzip -q sdk.zip && grep -o 'apigClient\.[a-zA-Z]*' apiGateway-js-sdk/apigClient.js | sort -u | tr '\n' ' '; echo; aws s3 sync apiGateway-js-sdk s3://$BUCKET/js/apiGateway-js-sdk --only-show-errors && aws s3 sync site s3://$BUCKET --only-show-errors && aws s3 ls s3://$BUCKET/ && aws s3 ls s3://$BUCKET/js/ | grep -v PRE
```

   - (1) Deploy stage `prod` và tạo SDK.
   - (2) SDK giờ có các hàm `conversationsPost`, `usersGet` và các hàm khác.
   - (3) Các trang mới và `amazon-cognito-identity.min.js` đã nằm trong bucket.

   ![Upload bản cuối](/images/6-identity/6.4-final-upload.png)

3. Sau khi xác nhận email, `confirm.html` chuyển người dùng về `/`. CloudFront cần biết trả về object nào cho đường dẫn này, nếu không S3 sẽ trả `AccessDenied`. Trong console CloudFront, mở distribution, chọn **Edit** ở **Settings** và nhập **Default root object** là `index.html` (1). Chọn **Save changes** và chờ **Last modified** hiện ngày giờ.

   ![Default root object](/images/6-identity/6.4-default-root.png)

4. Mở `https://<CloudFront domain>/signup.html`.
   - (1) Nhập tên người dùng, email bạn đọc được và mật khẩu tự chọn (tối thiểu 8 ký tự, có chữ hoa, chữ thường, số và ký tự đặc biệt).
   - (2) Chọn **Sign Up**.

   ![Đăng ký](/images/6-identity/6.4-signup.png)

5. Cognito gửi mã xác nhận qua email. Ở trang `confirm.html`, nhập tên người dùng và mã, sau đó đăng nhập ở trang chủ bằng tên người dùng và mật khẩu.

{{% notice tip %}}
Để có người trò chuyện cùng, hãy tạo thêm một người dùng từ CloudShell. Người dùng này chỉ cần có trong pool, không cần đăng nhập:

```bash
aws cognito-idp admin-create-user --user-pool-id $POOL_ID --username brian --user-attributes Name=email,Value=brian@example.com --message-action SUPPRESS --query User.Username --output text
```
{{% /notice %}}
