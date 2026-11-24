---
title : "Search expressions"
weight : 2
chapter : false
pre : " <b> 3.2 </b> "
---


Selecting metrics one by one does not scale. A **search expression** finds every metric that matches a query and graphs them together. New instances that match the query appear automatically.

1. On **Graphed metrics** choose **Clear graph**. On **Browse**, choose **All**, then **CWAgent**, then **ImageId, InstanceId, InstanceType** (1). Choose **Graph search** (2).

![CWAgent dimensions](/images/3-metrics/3.2-cwagent-dims.png)

2. On **Graphed metrics** a new expression `e1` appears with a `SEARCH(...)` call in **Details** (1). It graphs the memory usage of all three instances (2).

![Graph search](/images/3-metrics/3.2-graph-search.png)

3. Choose the pencil icon next to the expression and replace it with the expression below (1), then choose **Apply** (2).

   ```
   SEARCH('Namespace="CWAgent" MetricName="disk_used_percent"', 'Average', 300)
   ```

![Edit the search expression](/images/3-metrics/3.2-edit-disk.png)

   The graph now shows the root disk usage of the three instances (2), found by the namespace and metric name (1).

![disk_used_percent](/images/3-metrics/3.2-disk-result.png)

4. Search expressions also accept free text. Change the expression to the one below (1) and choose **Apply** (2).

   ```
   SEARCH('Namespace="CWAgent" used', 'Average', 300)
   ```

![Search by keyword](/images/3-metrics/3.2-edit-used.png)

   Every `CWAgent` metric whose name contains the word `used` is graphed: `mem_used_percent` and `disk_used_percent` (1).

![Keyword result](/images/3-metrics/3.2-used-result.png)
