---
title : "Create the S3 bucket"
weight : 1
chapter : false
pre : " <b> 2.1 </b> "
---

1. Open the **Amazon S3** console and choose **Create bucket**.
   - (1) **Bucket name**: a unique name, for example `fcj-chat-app-lab-<suffix>`. This guide uses `fcj-chat-app-lab-2611`.

   ![Bucket name](/images/2-static-website/2.1-bucket-name.png)

2. Keep the secure defaults:
   - (1) **Object Ownership**: **ACLs disabled (recommended)**.
   - (2) **Block all public access** stays selected.
   - Scroll down and choose **Create bucket**.

   ![Private bucket](/images/2-static-website/2.1-bucket-private.png)

3. Open **CloudShell**, download the source code and upload the `v1` site to the bucket:

   ```bash
   echo 'export BUCKET=fcj-chat-app-lab-2611' > ~/chat.env && source ~/chat.env
   git clone https://github.com/HoangGuruu/ws2-chatapp.git && cd ws2-chatapp
   aws s3 sync v1/Site s3://$BUCKET --exclude '*desktop.ini' --only-show-errors && aws s3 ls s3://$BUCKET/
   ```

   - (1) The sync command uploads the `v1/Site` folder.
   - (2) The bucket now holds `chat.html`, `chats.html` and the `css`, `fonts`, `js` folders.

   ![Upload v1](/images/2-static-website/2.1-upload-v1.png)

{{% notice tip %}}
The file `~/chat.env` keeps the variables of this workshop. If CloudShell restarts, run `source ~/chat.env` to load them again.
{{% /notice %}}
