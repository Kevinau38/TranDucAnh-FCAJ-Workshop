---
title : "Math expressions"
weight : 3
chapter : false
pre : " <b> 3.3 </b> "
---


**Metric math** calculates new time series from existing metrics or expressions, for example sums, averages, percentages or sorting.

1. Keep the search expression `e1` from section 3.2. Choose **Add math** (1), **Filter** (2), then **Top 10 by sum** (3).

![Add math](/images/3-metrics/3.3-add-math-top10.png)

2. A new expression `e2` with `SORT(METRICS(), SUM, DESC, 10)` is added (1).

![Top 10 by sum](/images/3-metrics/3.3-top10-added.png)

3. Edit `e2` and replace it with the expression below (1), then choose **Apply** (2). It sorts the results of `e1` by their average value, from high to low, and keeps only the first 3.

   ```
   SORT(e1, AVG, DESC, 3)
   ```

![Edit the math expression](/images/3-metrics/3.3-edit-sort.png)

4. Clear the check box of `e1` (1) so only the result of `e2` (2) is graphed. The legend now shows only the 3 series with the highest average, numbered in order (3).

![Top 3 by average](/images/3-metrics/3.3-sort-result.png)

{{% notice info %}}
See [Using math expressions with CloudWatch metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/using-metric-math.html) for all functions.
{{% /notice %}}
