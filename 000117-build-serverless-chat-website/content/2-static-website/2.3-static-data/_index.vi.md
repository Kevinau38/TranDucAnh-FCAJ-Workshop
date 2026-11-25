---
title : "Thêm dữ liệu tĩnh"
weight : 3
chapter : false
pre : " <b> 2.3 </b> "
---

Phiên bản `v2` của website đọc danh sách cuộc trò chuyện từ các file JSON trong thư mục `data`.

1. Upload `v2` trong CloudShell:

   ```bash
   cd ~/ws2-chatapp && aws s3 sync v2/Site s3://$BUCKET --exclude '*desktop.ini' && aws s3 ls s3://$BUCKET/data/ --recursive
   ```

   - (1) Lệnh sync upload các trang mới, `js/site.js` và file dữ liệu.
   - (2) `data/conversations.json` liệt kê các cuộc trò chuyện, `data/conversations/1.json` và `2.json` chứa tin nhắn.

   ![Upload v2](/images/2-static-website/2.3-upload-v2.png)

2. Tải lại `https://<distribution-domain>/chats.html`.
   - (1) Trang hiện hai cuộc trò chuyện **Brian** và **Frank**.

   ![Danh sách cuộc trò chuyện](/images/2-static-website/2.3-chats-static.png)

3. Chọn **Brian**.
   - (1) `chat.html#1` tải tin nhắn từ `data/conversations/1.json`.

   ![Cuộc trò chuyện tĩnh](/images/2-static-website/2.3-chat-static.png)

Nút **Send** chưa làm gì: website tĩnh không lưu được dữ liệu. Phần tiếp theo bạn thêm API.
