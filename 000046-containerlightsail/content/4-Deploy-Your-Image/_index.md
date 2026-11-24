---
title : "Deploy your image"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

You now replace the nginx image with your own image. Lightsail creates **version 2** of the deployment and keeps version 1, so you can roll back.

1. On the **Deployments** tab, choose **Modify your deployment**. In the `web` container, choose **Choose stored image** and select `:fcj-container.fcj-nginx.<number>`.

![Choose the stored image](/images/4/1-choose.png)

2. The **Image** field now shows your stored image. Keep the port `80` and the public endpoint `web`.

![Stored image selected](/images/4/2-selected.png)

3. Scroll down and choose **Save and deploy**.

![Save and deploy](/images/4/3-save.png)

4. The service shows **Deploying** with the new image. The old version keeps serving traffic until the new one passes its health check, so there is no downtime.

![Deploying version 2](/images/4/4-deploying.png)

5. After a few minutes, the status is **Running** and the container uses your image.

![Running](/images/4/5-running.png)

6. Under **Deployment versions**, version 2 is **Active** and version 1 is **Inactive**. To roll back, open the menu (⋮) of version 1 and deploy it again.

![Deployment versions](/images/4/6-versions.png)

7. Open the public domain again, or run the `curl` command of chapter 2. You see your page **Welcome to the Amazon Lightsail Container workshop**.

![Your page](/images/4/7-curl.png)
