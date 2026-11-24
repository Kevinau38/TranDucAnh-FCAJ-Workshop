---
title : "Deploy a public image"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

A **deployment** describes the containers that run on the service: the image, the open ports and which container receives the traffic of the public domain. Each time you save a deployment, Lightsail creates a new **version**.

1. When the service is **Ready**, open the **Deployments** tab and choose **Create your first deployment**.

![Create the first deployment](/images/2/1-ready.png)

2. Fill in the container:
   - **Container name**: `web`
   - **Image**: `nginx:stable-alpine`. Without a registry name, Lightsail pulls the image from Docker Hub.
   - Choose **Add open ports**, enter `80` and keep **HTTP**.

![Container settings](/images/2/2-container.png)

3. Under **Public endpoint**, choose `web`. The port is `80` and the health check path is `/`. Lightsail sends the traffic of the public domain to this container and checks it on this path. Choose **Save and deploy**.

![Public endpoint](/images/2/3-endpoint.png)

4. The status changes to **Deploying** (version 1). It takes about 3–5 minutes before it becomes **Running**.

![Deploying](/images/2/4-deploying.png)

5. Open the **Public domain** in your browser. You see the page **Welcome to nginx!**. You can also check it from CloudShell:

```bash
URL=$(aws lightsail get-container-services --region ap-southeast-1 \
  --service-name fcj-container --query 'containerServices[0].url' --output text)
echo $URL
curl -s $URL | grep -E '<title>|<h1>'
```

![nginx response](/images/2/5-curl-nginx.png)
