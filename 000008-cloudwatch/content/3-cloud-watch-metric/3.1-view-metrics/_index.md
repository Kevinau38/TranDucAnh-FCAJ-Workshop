---
title : "View metrics"
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---


1. Open the **CloudWatch** console. In the navigation pane, expand **Metrics** (1) and choose **Classic metrics** (2). Under **Custom namespaces** you see `CWAgent`, created by the CloudWatch agent (3). The `FCJ/CloudWatchWorkshop` namespace appears later, after section 4.3.

![Classic metrics](/images/3-metrics/3.1-classic-metrics.png)

2. Choose **1h** for the time range. In the search box enter `EC2` and press **Enter** (1), then choose **EC2 > Per-Instance Metrics** (2).

![EC2 per-instance metrics](/images/3-metrics/3.1-search-ec2.png)

3. In the search box enter `CPUUtilization` and press **Enter**. Sort by **Instance name** and select **FCJ-CloudWatch-Instance-A** and **FCJ-CloudWatch-Instance-B** (1). The graph shows both lines (2). Both instances go up and down in cycles, with A busier than B.

![CPU of two instances](/images/3-metrics/3.1-cpu-two-instances.png)

{{% notice info %}}
Your account can contain other instances. Only select the rows whose name starts with `FCJ-CloudWatch-`.
{{% /notice %}}

4. Clear Instance B. Remove the `CPUUtilization` filter, search for `EBSWriteBytes` and select the row of **Instance-A** (1). The graph now shows CPU and disk writes of the same instance (2). The two lines move together, because the workload writes to disk right after its CPU-busy phase.

![CPU and EBS write bytes](/images/3-metrics/3.1-cpu-and-ebs.png)

5. The two metrics use different units (percent and bytes), so the CPU line looks flat. Open the **Graphed metrics** tab and, on the `EBSWriteBytes` row, choose the right arrow in the **Y axis** column (1). The metric moves to a second Y axis on the right (2) and both lines become readable.

![Right Y axis](/images/3-metrics/3.1-right-axis.png)
