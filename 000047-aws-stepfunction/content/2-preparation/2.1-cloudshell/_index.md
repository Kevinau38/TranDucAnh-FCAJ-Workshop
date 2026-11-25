---
title : "Open AWS CloudShell"
weight : 1
chapter : false
pre : " <b> 2.1 </b> "
---

AWS CloudShell is a browser-based shell that already has the AWS CLI, the AWS SAM CLI and Node.js installed. It uses the permissions of the user signed in to the console, so you do not need an IAM user or access keys.

1. Sign in to the AWS Management Console and switch to the Region **Asia Pacific (Tokyo) ap-northeast-1**.
2. Choose the **CloudShell** icon on the navigation bar.
3. When the prompt appears, check the tools:

   ```bash
   sam --version; node --version; aws --version
   ```

   - (1) AWS SAM CLI, Node.js and AWS CLI v2 are ready.

   ![CloudShell](/images/2-preparation/2.1-cloudshell.png)

{{% notice note %}}
Files in your CloudShell home folder are kept between sessions in the same Region, so you can close the tab and continue later.
{{% /notice %}}
