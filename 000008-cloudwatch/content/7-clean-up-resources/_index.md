---
title : "Clean up resources"
weight : 7
chapter : false
pre : " <b> 7. </b> "
---


Delete the resources in this order. The dashboard, alarm and SNS topic were created in the console, so the stack does not delete them.

1. **Dashboard**: in **Dashboards**, select **FCJ-CloudWatch-Workshop**, choose **Delete** and confirm (1).

![Delete dashboard](/images/7-cleanup/7-delete-dashboard.png)

2. **Alarm**: in **Alarms**, select **PythonAppErrorAlarm** (1), choose **Actions**, **Delete** (2) and confirm.

![Delete alarm](/images/7-cleanup/7-delete-alarm.png)

3. **SNS topic**: open the **Amazon SNS** console, **Topics**, select **FCJ-CloudWatch-Alarm-Topic**, choose **Delete**, type `delete me` (1) and choose **Delete** (2). Then open **Subscriptions** and delete the email subscription of that topic if it is still listed.

![Delete SNS topic](/images/7-cleanup/7-delete-topic.png)

4. **Stack**: in the **CloudFormation** console select **FCJ-CloudWatch-Workshop**, choose **Delete**, type the stack name (1) and choose **Delete stack** (2). The stack deletes the instances, the VPC, the IAM role and the log group with its metric filter. Wait until the stack disappears from the **Active** list, about 3 minutes.

![Delete stack](/images/7-cleanup/7-delete-stack.png)

{{% notice info %}}
Metrics are not deleted. Custom metrics such as `PythonAppErrorCount` and the `CWAgent` metrics stop receiving data and expire automatically; you are not charged for metrics that receive no new data.
{{% /notice %}}
