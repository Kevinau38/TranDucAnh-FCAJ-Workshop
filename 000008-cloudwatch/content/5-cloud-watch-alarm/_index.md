---
title : "CloudWatch Alarms"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---


In this section you create an alarm on `PythonAppErrorCount`. When the application writes more than 5 errors in one minute, the alarm goes to **In alarm** and sends an email through Amazon SNS.

#### Create the alarm

1. In the CloudWatch navigation pane choose **Alarms**, **All alarms**, then **Create alarm** (1).

![Create alarm](/images/5-alarm/5-create-alarm.png)

2. Keep **Metrics** and **Classic**, then choose **Select metric** (1).

![Select metric](/images/5-alarm/5-select-metric.png)

3. Choose the custom namespace **FCJ/CloudWatchWorkshop** (1), then **Metrics with no dimensions**.

![Namespace](/images/5-alarm/5-namespace.png)

4. Select **PythonAppErrorCount** (1) and choose **Select metric** (2).

![PythonAppErrorCount](/images/5-alarm/5-metric-selected.png)

5. Change **Statistic** to **Sum** (1) and **Period** to **1 minute** (2). The graph now shows the number of errors per minute.

![Statistic and period](/images/5-alarm/5-stat-period.png)

6. Under **Conditions** keep **Static** and **Greater** (1), enter `5` as the threshold (2) and choose **Next** (3). The red dashed line in the preview is the threshold.

![Threshold](/images/5-alarm/5-threshold.png)

7. Under **Notification**, keep **In alarm** and select **Create new topic** (1). Enter the topic name `FCJ-CloudWatch-Alarm-Topic` (2), your email address (3), and choose **Create topic** (4).

![Create SNS topic](/images/5-alarm/5-new-topic.png)

8. The new topic is selected (1) with your email as an endpoint (2). Scroll down and choose **Next**.

![Topic created](/images/5-alarm/5-topic-created.png)

9. Enter **Alarm name** `PythonAppErrorAlarm` (1) and a description, for example `More than 5 ERROR lines from python-app in 1 minute` (2). Choose **Next**.

![Alarm name](/images/5-alarm/5-alarm-name.png)

10. Review the alarm, check the notification (1) and choose **Create alarm** (2).

![Create alarm](/images/5-alarm/5-preview-create.png)

11. The alarm is created (1). Its state is **Insufficient data** until it has enough data points, and a banner reminds you that the SNS subscription is pending confirmation.

![Alarm created](/images/5-alarm/5-alarm-created.png)

#### Confirm the email subscription

12. Open the email **AWS Notification - Subscription Confirmation** from `no-reply@sns.amazonaws.com` and choose **Confirm subscription** (1).

![Confirmation email](/images/5-alarm/5-email-confirm.png)

13. On the page that opens choose **Confirm Subscription** (1).

![Confirm subscription](/images/5-alarm/5-sns-confirm.png)

14. The subscription is confirmed (1). SNS only sends notifications to confirmed subscriptions.

![Subscription confirmed](/images/5-alarm/5-sns-confirmed.png)

#### Trigger the alarm

15. `logger.py` usually writes 2 to 5 errors per minute, so the alarm stays **OK** most of the time. Simulate an outage: open a Session Manager session to Instance A again and write 30 error lines at once (1):

   ```bash
   for i in $(seq 1 30); do logger -t python-app "ERROR GET /api/orders failed: simulated outage $i"; done; echo done
   ```

![Simulate errors](/images/5-alarm/5-simulate-errors.png)

16. Wait 1 to 3 minutes and open the alarm. The graph shows the spike above the threshold (1) and the state timeline turns red for **In alarm** (2). After the next minute the error count drops again and the alarm returns to **OK**.

![Alarm triggered](/images/5-alarm/5-alarm-triggered.png)

17. Check your mailbox. SNS sent the email **ALARM: "PythonAppErrorAlarm" in ...** (1). It contains the reason, the value that crossed the threshold (2) and the state change `OK -> ALARM` (3).

![Alarm email](/images/5-alarm/5-alarm-email.png)
