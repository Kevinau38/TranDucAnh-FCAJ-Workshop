---
title : "Tạo bucket S3"
weight : 1
chapter : false
pre : " <b> 2.1 </b> "
---

1. Mở console **Amazon S3**, chọn **Create bucket**.
   - (1) **Bucket name**: một tên duy nhất, ví dụ `fcj-chat-app-lab-<hậu tố>`. Bài này dùng `fcj-chat-app-lab-2611`.

   ![Tên bucket](/images/2-static-website/2.1-bucket-name.png)

2. Giữ các thiết lập mặc định an toàn:
   - (1) **Object Ownership**: **ACLs disabled (recommended)**.
   - (2) Giữ chọn **Block all public access**.
   - Cuộn xuống, chọn **Create bucket**.

   ![Bucket private](/images/2-static-website/2.1-bucket-private.png)

3. Mở **CloudShell**, tải mã nguồn và upload website `v1` lên bucket:

   ```bash
   echo 'export BUCKET=fcj-chat-app-lab-2611' > ~/chat.env && source ~/chat.env
   git clone https://github.com/HoangGuruu/ws2-chatapp.git && cd ws2-chatapp
   aws s3 sync v1/Site s3://$BUCKET --exclude '*desktop.ini' --only-show-errors && aws s3 ls s3://$BUCKET/
   ```

   - (1) Lệnh sync upload thư mục `v1/Site`.
   - (2) Bucket đã có `chat.html`, `chats.html` và các thư mục `css`, `fonts`, `js`.

   ![Upload v1](/images/2-static-website/2.1-upload-v1.png)

{{% notice tip %}}
File `~/chat.env` lưu các biến của bài. Nếu CloudShell khởi động lại, chạy `source ~/chat.env` để nạp lại.
{{% /notice %}}
