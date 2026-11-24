---
title : "Deploy the instance"
weight : 1
chapter : false
pre : " <b> 3.1. </b> "
---

1. In the Lightsail console, choose **Instances**, then **Create instance**. Keep **Singapore, Zone A**.

2. Under **Select a platform**, choose **Linux apps**. Under **Select a blueprint**, choose **PrestaShop**.

![PrestaShop blueprint](/images/3/1-blueprint.png)

{{% notice info %}}
The PrestaShop blueprint is labeled **Originally packaged by Bitnami (no longer maintained)**. Bitnami stopped updating these images: after May 19, 2026 they receive no new updates, and from May 19, 2027 you can no longer create new instances from them. For a long-lived shop, plan to install PrestaShop on an OS blueprint, the same way you installed WordPress.
{{% /notice %}}

3. Under **Select a size**, choose the **USD 12** plan (2 GB memory). With 1 GB, Lightsail warns that PrestaShop needs a larger plan. PrestaShop does not support **IPv6-only** plans, so keep **Dual-stack**.

![PrestaShop plan](/images/3/2-plan.png)

4. Under **Instance name**, enter `fcj-prestashop`. Add the tag `Project` = `fcj-lightsail`. Choose **Create instance**.

![Name and tag](/images/3/3-name-tag.png)

5. Wait 3–5 minutes for the instance to finish its first start.
