---
title : "Create alarms"
weight : 7
chapter : false
pre : " <b> 7. </b> "
---

Lightsail **alarms** watch a metric and warn you when it crosses a threshold. On burstable plans, CPU usage above the **sustainable zone** for a long time uses up the CPU burst capacity, and the instance slows down. A CPU alarm warns you before that happens.

1. Open the instance `fcj-wordpress-2gb`, choose the **Metrics** tab, and choose the metric **CPU utilization**.

2. Under **Alarms for: CPU utilization**, choose **Create alarm**.

3. Set the rule to **Greater than or equal to** **70** percent, **2** times within the last **20 minutes**. The blue sentence under the fields summarizes the rule.

4. Under **Also notify me by**, you can select **Email** to receive notifications. Email notifications need a verified email contact for the Region; you add it on the **Alarm notifications** page in the left menu. Without it, the alarm only shows a banner in the Lightsail console. Choose **Save**.

![Create a CPU alarm](/images/7/1-alarm.png)

5. The alarm is created and is in the **OK** state.

![Alarm in OK state](/images/7/2-alarm-ok.png)

{{% notice tip %}}
You can add up to two alarms per metric. Useful alarms for this lab are **CPU burst capacity (percentage)** for the instances and **CPU utilization** or **Free storage space** for the database.
{{% /notice %}}
