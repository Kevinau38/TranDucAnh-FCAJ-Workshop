---
title : "Review the security group"
weight : 2
chapter : false
pre : " <b> 2.2 </b> "
---


#### Review the security group

The original version of this lab opened SSH (port 22) to your IP address. Because you will connect with **Session Manager**, the instance does not need any inbound port.

1. Open the [EC2 console](https://console.aws.amazon.com/ec2/home) → **Security Groups** and search for `SID-instance-sg`.
2. Select it and check the **Details** tab: **Inbound rules count** is **0 Permission entries**. The only outbound rule allows the instance to reach AWS service endpoints over HTTPS.

![Security group without inbound rules](/images/2-prerequisite/2.2-security-group.png)

{{% notice tip %}}
Session Manager uses an outbound connection from the SSM Agent to the Systems Manager service, so you can close every inbound port and still get a shell on the instance.
{{% /notice %}}
