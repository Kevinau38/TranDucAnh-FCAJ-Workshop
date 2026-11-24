---
title : "CloudWatch Logs Insights"
weight : 2
chapter : false
pre : " <b> 4.2 </b> "
---


First you start the sample application on Instance A so that it writes log lines, then you query them with **Logs Insights**.

#### Start the sample application

1. Open the **EC2** console, choose **Instances**, select **FCJ-CloudWatch-Instance-A** (1) and choose **Connect** (2).

![Select Instance-A](/images/4-logs/4.2-select-instance.png)

2. Select **SSM Session Manager** (1) and choose **Connect** (2).

![Session Manager](/images/4-logs/4.2-connect-ssm.png)

3. In the terminal, view the script, start it in the background (1) and check that it is running (2):

   ```bash
   cat /opt/fcj/logger.py
   nohup python3 /opt/fcj/logger.py > /dev/null 2>&1 &
   ps aux | grep [l]ogger.py
   ```

![Start logger.py](/images/4-logs/4.2-logger-start.png)

   The script writes a line every 1 to 4 seconds: mostly `INFO`, some `WARNING` and about 15% `ERROR`. Keep it running until the end of the lab.

4. Look at the log file. Lines from `python-app` appear between the system messages (1)(2). Press **Ctrl+C** to stop `tail`.

   ```bash
   sudo tail -f /var/log/messages
   ```

![tail /var/log/messages](/images/4-logs/4.2-tail.png)

#### Query with Logs Insights

5. In the CloudWatch navigation pane choose **Logs**, **Logs Insights**. Open the log group selector, filter by `/fcj/cloudwatch` (1) and select **/fcj/cloudwatch-workshop/var/log/messages** (2). Close the selector.

![Choose the log group](/images/4-logs/4.2-choose-log-group.png)

6. Check that the log group is selected (1), enter the query below (2) and choose **Run query** (3).

   ```
   fields @timestamp, @logStream, @message
   | filter @message like /python-app/
   | sort @timestamp desc
   | limit 20
   ```

![Logs Insights query](/images/4-logs/4.2-query.png)

7. The histogram and the table show the latest 20 lines from `python-app` (1)(2).

![Query results](/images/4-logs/4.2-results.png)

8. Logs Insights can also parse and aggregate. Run the query below (1)(2). It extracts the level word after `python-app[pid]:` and counts the lines per level.

   ```
   filter @message like /python-app/
   | parse @message "python-app[*]: * " as pid, level
   | stats count(*) as events by level
   ```

![stats query](/images/4-logs/4.2-stats-query.png)

9. Open the **Visualization** tab and change **Graph type** to **Bar** (1). You see the number of `ERROR`, `WARNING` and `INFO` lines (2).

![Bar chart](/images/4-logs/4.2-stats-bar.png)
