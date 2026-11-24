---
title : "Secure the applications"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

Each Lightsail instance has a **firewall** that controls inbound traffic. By default it allows HTTP (port 80) and SSH (port 22) from any IPv4 or IPv6 address. Visitors need HTTP, but SSH open to the whole internet is a common target for password-guessing bots. You restrict SSH so that only the **Lightsail browser SSH client** can connect.

{{% notice warning %}}
After this change, the `lsssh.sh` script in CloudShell can no longer connect to the instance. Do this chapter after you have finished chapters 2 and 3. If you need CloudShell access again, add your source IP back to the SSH rule.
{{% /notice %}}

1. Open the instance `fcj-wordpress` and choose the **Networking** tab. The **Firewall rules** table shows the default rules.

![Default firewall rules](/images/4/1-firewall.png)

2. On the **SSH** rule, choose the edit (pencil) icon.

3. Under **Source IP addresses**, choose **Remove** for both **Anywhere IPv4** and **Anywhere IPv6**. Keep **Lightsail browser SSH client** with **Enable IPv4** and **Enable IPv6** selected. Choose **Update rule**.

![Edit the SSH rule](/images/4/2-edit-rule.png)

4. The SSH rule now shows **Lightsail browser SSH only**.

![SSH restricted](/images/4/3-ssh-only.png)

5. On the **Connect** tab, choose **Connect using SSH** to check that browser SSH still works.

![Browser SSH](/images/4/4-browser-ssh.png)

6. Repeat steps 1–4 for `fcj-prestashop`.

{{% notice tip %}}
Other good practices for Lightsail instances: keep the operating system updated (`sudo apt update && sudo apt upgrade`), open only the ports you need, and use HTTPS with a TLS certificate for any site that has a login page.
{{% /notice %}}
