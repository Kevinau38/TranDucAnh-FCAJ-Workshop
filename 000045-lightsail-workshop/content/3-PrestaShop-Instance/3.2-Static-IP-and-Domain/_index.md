---
title : "Attach a static IP and set the domain"
weight : 2
chapter : false
pre : " <b> 3.2. </b> "
---

PrestaShop also stores its own address and redirects every visitor to it. After you attach a static IP, you tell PrestaShop to use that address.

1. Create and attach a static IP named `fcj-prestashop-ip`. You can repeat the steps of [2.2](../../2-wordpress-instance/2.2-static-ip/) in the console, or run these commands in CloudShell:

```bash
aws lightsail allocate-static-ip --static-ip-name fcj-prestashop-ip
aws lightsail attach-static-ip --static-ip-name fcj-prestashop-ip --instance-name fcj-prestashop
```

2. The **Instances** page now shows a pin icon next to the public IP of both instances: they use static IPs.

![Instances with static IPs](/images/3/4-instances.png)

3. In CloudShell, set the PrestaShop domain to the static IP with the Bitnami tool `configure_app_domain`, then check the redirect. The last line must be `302 http://<static-ip>/`.

```bash
PSIP=$(aws lightsail get-static-ip --static-ip-name fcj-prestashop-ip --query staticIp.ipAddress --output text)
./lsssh.sh fcj-prestashop "sudo /opt/bitnami/configure_app_domain --domain $PSIP"
./lsssh.sh fcj-prestashop "curl -s -o /dev/null -w '%{http_code} %{redirect_url}\n' http://localhost/"
```

{{% notice note %}}
`configure_app_domain` may print `Job for bitnami-config-hostname.service failed`. The domain is still updated; the `302` redirect to the static IP confirms it.
{{% /notice %}}

4. Open `http://<static-ip>/` in your browser. The PrestaShop demo store is running.

![PrestaShop store](/images/3/5-storefront.png)

{{% notice tip %}}
The PrestaShop administrator user name and password are stored on the instance. To read them, run `./lsssh.sh fcj-prestashop 'sudo cat /home/bitnami/bitnami_credentials'` in CloudShell. Do not share the output.
{{% /notice %}}
