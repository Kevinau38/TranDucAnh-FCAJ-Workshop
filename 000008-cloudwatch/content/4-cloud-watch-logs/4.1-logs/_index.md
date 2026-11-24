---
title : "Log groups and log streams"
weight : 1
chapter : false
pre : " <b> 4.1 </b> "
---


1. In the CloudWatch navigation pane, expand **Logs** and choose **Log Management** (1). Filter by `/fcj/cloudwatch-workshop` (2) and open the log group **/fcj/cloudwatch-workshop/var/log/messages** (3).

![Log groups](/images/4-logs/4.1-log-groups.png)

2. The **Log streams** tab lists one stream per instance, named by instance ID (1). Open one of them.

![Log streams](/images/4-logs/4.1-log-streams.png)

3. You see the system log of the instance: `systemd`, `amazon-ssm-agent` and other services (1). Use the filter box to search, for example `ssm`.

![Log events](/images/4-logs/4.1-log-events.png)

4. Go back to the log group. By default logs never expire, which costs money over time. Choose **Actions** (1), then **Edit retention setting** (2).

![Edit retention setting](/images/4-logs/4.1-actions-retention.png)

5. Set **Expire events after** to **1 week (7 days)** (1) and choose **Save** (2).

![Retention setting](/images/4-logs/4.1-retention-dialog.png)

6. A banner confirms the change (1).

![Retention updated](/images/4-logs/4.1-retention-saved.png)
