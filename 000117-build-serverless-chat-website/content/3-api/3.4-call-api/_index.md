---
title : "Deploy the API and call it from the site"
weight : 4
chapter : false
pre : " <b> 3.4 </b> "
---

1. Choose **Deploy API**:
   - (1) **Stage**: **New stage**, **Stage name**: `prod`.
   - (2) Choose **Deploy**.

   ![Deploy stage](/images/3-api/3.4-deploy-stage.png)

2. On the **Stages** page:
   - (1) Stage `prod` is created.
   - (2) Copy the **Invoke URL**.

   ![Invoke URL](/images/3-api/3.4-invoke-url.png)

3. Save the API in CloudShell, write the URL into `v3/Site/js/config.js` and upload `v3`:

   ```bash
   echo 'export API_ID=<api-id>' >> ~/chat.env
   echo 'export API_URL=https://<api-id>.execute-api.ap-northeast-1.amazonaws.com/prod' >> ~/chat.env && source ~/chat.env
   cd ~/ws2-chatapp && sed -i "s#^ChatApp.apiEndpoint.*#ChatApp.apiEndpoint = '$API_URL';#" v3/Site/js/config.js
   aws s3 cp v3/Site/js/ s3://$BUCKET/js/ --recursive --exclude '*desktop.ini' --only-show-errors
   ```

   - (1) `config.js` now points to your API.
   - (2) `config.js` and `site.js` are uploaded.

   ![Config and upload](/images/3-api/3.4-config-upload.png)

4. Reload `chats.html`. The list looks the same, but the data now comes from `GET /conversations` of the API, not from the JSON file.
