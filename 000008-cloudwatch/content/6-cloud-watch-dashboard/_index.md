---
title : "CloudWatch Dashboards"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---


A **dashboard** puts the graphs and alarms you care about on one page.

1. In **Alarms**, select **PythonAppErrorAlarm** (1), choose **Actions** (2), then **Add to dashboard** (3).

![Add to dashboard](/images/6-dashboard/6-add-to-dashboard.png)

2. Choose **Create new**, enter `FCJ-CloudWatch-Workshop` (1) and choose **Create** (2).

![Create dashboard](/images/6-dashboard/6-create-dashboard.png)

3. The new dashboard is selected (1). Keep the **Line** widget type and choose **Add to dashboard** (2).

![Add to dashboard](/images/6-dashboard/6-add-widget.png)

4. A banner confirms the widget was added (1). In this screenshot the alarm is **In alarm** (2), because of the errors simulated in section 5. Choose **View dashboard**.

![Widget added](/images/6-dashboard/6-in-alarm.png)

5. Add a second widget. Choose **+** (Add widget) at the top right, keep **Line** (1) and choose **Next** (2).

![Add widget](/images/6-dashboard/6-widget-type.png)

6. Rename the graph to `Memory used (%)` with the pencil icon (1). Search for `mem_used_percent` (2), choose **Graph search**, then choose **Create widget** (3).

![Memory widget](/images/6-dashboard/6-memory-widget.png)

7. The dashboard now shows the alarm widget (1), red while the alarm is in alarm, and the memory widget of the three instances (2). Choose **Save** (3).

![Dashboard](/images/6-dashboard/6-dashboard.png)

{{% notice warning %}}
Changes to a dashboard are not saved automatically. Always choose **Save** after adding or moving widgets.
{{% /notice %}}
