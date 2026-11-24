---
title : "Connect with Session Manager"
weight : 4
chapter : false
pre : " <b> 2.4 </b> "
---


#### Connect to the instance with Session Manager

1. In the EC2 console, open **Instances**, select `SID-security-instance` and choose **Connect**.
2. On the **Session Manager** tab (shown as **SSM Session Manager**), choose **Connect**.

![Connect with Session Manager](/images/2-prerequisite/2.4-connect.png)

3. A browser terminal opens. Load the lab variables and list the bucket:

```bash
cd ~ && source /etc/profile.d/sid-lab.sh && echo $bucket && aws s3 ls s3://$bucket --recursive
```

The instance prints the bucket name and the test object `app1/file1`.

![Session Manager shell](/images/2-prerequisite/2.4-shell.png)

{{% notice tip %}}
If the session is closed later, reconnect and run `cd ~ && source /etc/profile.d/sid-lab.sh` again before the next commands. The file sets `$bucket` and the default Region for the AWS CLI.
{{% /notice %}}
