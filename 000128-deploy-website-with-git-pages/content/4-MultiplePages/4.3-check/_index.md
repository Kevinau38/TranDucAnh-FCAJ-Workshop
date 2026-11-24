---
title : "Check the result"
weight : 3
chapter : false
pre : " <b> 4.3 </b> "
---


1. Choose the **Actions** tab, then the **Create static.yml** run. Wait until **Status** is **Success** (1). The **deploy** job shows the site address (2).

![Static HTML run](/images/4-MultiplePages/4.3-run-success.png)

{{% notice note %}}
The run summary may show a warning that **Node.js 20 is deprecated** and that some actions are forced to run on Node.js 24. This is a notice from GitHub about the action versions in the starter workflow; the deployment still succeeds. You can ignore it in this lab.
{{% /notice %}}

2. Open `https://<username>.github.io/workshop1/`. The site looks the same as the user site, but it is served from the `workshop1` repository.

![Project site workshop1](/images/4-MultiplePages/4.3-site.png)
