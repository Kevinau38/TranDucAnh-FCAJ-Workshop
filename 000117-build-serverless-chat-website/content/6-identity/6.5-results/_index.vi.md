---
title : "Kiểm tra kết quả"
weight : 5
chapter : false
pre : " <b> 6.5 </b> "
---

1. Sau khi đăng nhập, trang **people.html** liệt kê người dùng trong pool (1). Chọn **Start Chat** cạnh người dùng còn lại.

   ![Người dùng](/images/6-identity/6.5-people.png)

2. Gửi một tin nhắn. Tin nhắn hiện với tên người dùng của bạn, không còn là `Student` (1).

   ![Chat với Cognito](/images/6-identity/6.5-chat-cognito.png)

3. Chọn **Conversations**. Cuộc trò chuyện mới với người dùng kia đã có trong danh sách (1).

   ![Danh sách cuộc trò chuyện](/images/6-identity/6.5-conversations.png)

4. Gọi API không kèm token, sau đó xem tin nhắn trong DynamoDB (thay `kevin` bằng tên người dùng của bạn):

```bash
curl -s -w '\nHTTP %{http_code}\n' $API_URL/conversations; aws dynamodb scan --table-name Chat-Messages --query 'Items[?Sender.S==`kevin`].[Sender.S,Message.S]' --output text
```

   - (1) Request không có header `Authorization`.
   - (2) API Gateway từ chối với `401 Unauthorized`, trước khi Lambda function nào chạy.
   - (3) Người gửi tin nhắn là tên người dùng lấy từ ID token.

   ![Unauthorized](/images/6-identity/6.5-unauthorized.png)
