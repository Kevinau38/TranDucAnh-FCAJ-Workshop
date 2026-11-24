---
title : "Check the result"
weight : 3
chapter : false
pre : " <b> 3.3 </b> "
---


1. Choose the **Actions** tab. GitHub created a **pages build and deployment** run for your push. Wait until it shows a green check mark, about one minute.

![pages build and deployment](/images/3-SinglePage/3.3-actions.png)

2. Open `https://<username>.github.io/` in your browser. The site shows the title and **My First Post**.

![User site](/images/3-SinglePage/3.3-site.png)

{{% notice info %}}
If you see a 404 page, wait one or two minutes and refresh. If the page has no styling, check that you built the site with `--baseURL https://<username>.github.io/`.
{{% /notice %}}
