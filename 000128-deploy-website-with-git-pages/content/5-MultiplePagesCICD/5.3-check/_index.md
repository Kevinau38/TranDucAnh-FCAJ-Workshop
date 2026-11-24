---
title : "Check the result"
weight : 3
chapter : false
pre : " <b> 5.3 </b> "
---


1. Choose the **Actions** tab. The run **Create hugo.yml** started from your commit (1).

![Hugo workflow run](/images/5-MultiplePagesCICD/5.3-runs.png)

2. Open the run. Wait until **Status** is **Success** (1). This time there are two jobs: **build** runs Hugo, then **deploy** publishes the result (2).

![build and deploy](/images/5-MultiplePagesCICD/5.3-run-detail.png)

{{% notice note %}}
The run summary may show a warning that **Node.js 20 is deprecated** and that some actions are forced to run on Node.js 24. This is a notice from GitHub about the action versions in the starter workflow; the deployment still succeeds. You can ignore it in this lab.
{{% /notice %}}

3. Open `https://<username>.github.io/workshop2/`.

![Project site workshop2](/images/5-MultiplePagesCICD/5.3-site.png)
