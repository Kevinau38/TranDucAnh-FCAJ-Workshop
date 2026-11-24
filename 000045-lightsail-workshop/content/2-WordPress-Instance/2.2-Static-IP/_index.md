---
title : "Attach a static IP"
weight : 2
chapter : false
pre : " <b> 2.2. </b> "
---

The public IPv4 address that Lightsail gives an instance changes when you stop and start the instance. WordPress saves its own URL in the database, so you give the instance a **static IP** before you install WordPress. A static IP is free while it is attached to an instance.

1. In the left menu, choose **Networking**, then **Create static IP**.

![Networking page](/images/2/6-networking.png)

2. Under **Attach to an instance**, choose `fcj-wordpress`. Under **Identify your static IP**, enter `fcj-wordpress-ip`. Choose **Create**.

![Attach the static IP](/images/2/7-attach.png)

3. The static IP is created and attached. Note the address: it is the address of your blog.

![Static IP](/images/2/8-static-ip.png)

{{% notice warning %}}
A static IP that is **not attached** to an instance is billed by the hour. Delete it if you detach it and no longer need it.
{{% /notice %}}
