---
title : "Metric filters"
weight : 3
chapter : false
pre : " <b> 4.3 </b> "
---


A **metric filter** counts log events that match a pattern and publishes the count as a CloudWatch metric. You use it to count the `ERROR` lines of `python-app`, and alarm on that metric in section 5.

1. Open the log group **/fcj/cloudwatch-workshop/var/log/messages**, choose **Actions** (1), then **Create metric filter** (2).

![Create metric filter](/images/4-logs/4.3-actions.png)

2. In **Filter pattern** enter the pattern below (1). Both terms must appear in a log event for it to match.

   ```
   "python-app" "ERROR"
   ```

![Filter pattern](/images/4-logs/4.3-pattern.png)

3. Under **Test pattern**, select the log stream of Instance A in **Select log data to test** (1) and choose **Test pattern** (2).

![Test data](/images/4-logs/4.3-test-data.png)

4. **Results** shows how many sample events match (1) and lists them (2). Only `ERROR` lines from `python-app` match. Choose **Next** (3).

![Test results](/images/4-logs/4.3-test-results.png)

5. Enter **Filter name** `PythonAppErrors` (1) and **Metric namespace** `FCJ/CloudWatchWorkshop` (2).

![Filter name and namespace](/images/4-logs/4.3-filter-name.png)

6. Enter the metric details:
   - **Metric name**: `PythonAppErrorCount` (1)
   - **Metric value**: `1` (2), each matching line adds 1
   - **Default value**: `0` (3), so the metric is 0 instead of missing when there is no error
   - **Unit**: `Count` (4)

![Metric details](/images/4-logs/4.3-metric-details.png)

7. Leave **Dimensions** empty and choose **Next** (1).

![Next](/images/4-logs/4.3-dimensions-next.png)

8. Review and choose **Create metric filter** (1).

![Create metric filter](/images/4-logs/4.3-review.png)

9. The filter is created (1) and listed on the **Metric filters** tab with its namespace and metric (2). After a minute the metric appears in **Metrics** under **FCJ/CloudWatchWorkshop**.

![Metric filter created](/images/4-logs/4.3-created.png)

{{% notice info %}}
A metric filter only processes log events that arrive after it was created. It does not count older events.
{{% /notice %}}
