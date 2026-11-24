---
title : "Dynamic labels"
weight : 4
chapter : false
pre : " <b> 3.4 </b> "
---


The legend of a search expression often contains long, hard to read names. **Dynamic labels** build the label of every series from its properties, and update automatically.

1. Choose **Clear graph**. On **Browse**, choose **CWAgent**, then **ImageId, InstanceId, InstanceType, exe, process_name** (1). These are the per-process (`procstat`) metrics of the CloudWatch agent process.

![procstat dimensions](/images/3-metrics/3.4-cwagent-dims.png)

2. Search for `memory_rss` (1) and choose **Graph search** (2). Three rows match, one per instance (3).

![procstat_memory_rss](/images/3-metrics/3.4-memory-rss.png)

3. On **Graphed metrics**, choose **Add dynamic label**, **Properties** (1), then **Dim.DimName - dimension value** (2).

![Add dynamic label](/images/3-metrics/3.4-dynamic-label-menu.png)

4. The label becomes `[dimension: ${PROP('Dim.DimName')}]` (1). `DimName` is only a placeholder, so the legend shows `--` (2).

![Placeholder label](/images/3-metrics/3.4-dimname-label.png)

5. Choose the pencil icon in the **Label** column, replace the label with the line below (1) and choose **Apply** (2).

   ```
   ${PROP('Dim.process_name')} - ${PROP('Dim.InstanceId')} - ${PROP('MetricName')}
   ```

![Edit the label](/images/3-metrics/3.4-edit-label.png)

6. Every series now has a clear label: process name, instance and metric name (1).

![Dynamic labels](/images/3-metrics/3.4-label-result.png)
