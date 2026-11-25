---
title : "Add static data"
weight : 3
chapter : false
pre : " <b> 2.3 </b> "
---

Version `v2` of the site reads the conversations from JSON files in the `data` folder.

1. Upload `v2` in CloudShell:

   ```bash
   cd ~/ws2-chatapp && aws s3 sync v2/Site s3://$BUCKET --exclude '*desktop.ini' && aws s3 ls s3://$BUCKET/data/ --recursive
   ```

   - (1) The sync uploads the new pages, `js/site.js` and the data files.
   - (2) `data/conversations.json` lists the conversations, `data/conversations/1.json` and `2.json` hold the messages.

   ![Upload v2](/images/2-static-website/2.3-upload-v2.png)

2. Reload `https://<distribution-domain>/chats.html`.
   - (1) The page lists two conversations, **Brian** and **Frank**.

   ![Conversation list](/images/2-static-website/2.3-chats-static.png)

3. Choose **Brian**.
   - (1) `chat.html#1` loads the messages from `data/conversations/1.json`.

   ![Static conversation](/images/2-static-website/2.3-chat-static.png)

The **Send** button does nothing yet: a static website cannot save data. You add an API in the next part.
