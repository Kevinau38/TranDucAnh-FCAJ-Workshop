---
title : "Math expression"
weight : 3
chapter : false
pre : " <b> 3.3 </b> "
---


**Metric math** tính ra chuỗi số mới từ các metric hoặc expression có sẵn, ví dụ tổng, trung bình, phần trăm hay sắp xếp.

1. Giữ search expression `e1` từ phần 3.2. Chọn **Add math** (1), **Filter** (2), rồi **Top 10 by sum** (3).

![Add math](/images/3-metrics/3.3-add-math-top10.png)

2. Expression mới `e2` với `SORT(METRICS(), SUM, DESC, 10)` được thêm vào (1).

![Top 10 by sum](/images/3-metrics/3.3-top10-added.png)

3. Sửa `e2` thành expression dưới đây (1) rồi chọn **Apply** (2). Expression sắp xếp các kết quả của `e1` theo giá trị trung bình từ cao xuống thấp và chỉ giữ 3 kết quả đầu.

   ```
   SORT(e1, AVG, DESC, 3)
   ```

![Sửa math expression](/images/3-metrics/3.3-edit-sort.png)

4. Bỏ chọn `e1` (1) để chỉ vẽ kết quả của `e2` (2). Chú thích giờ chỉ còn 3 chuỗi có trung bình cao nhất, đánh số theo thứ tự (3).

![Top 3 theo trung bình](/images/3-metrics/3.3-sort-result.png)

{{% notice info %}}
Xem [Using math expressions with CloudWatch metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/using-metric-math.html) để biết mọi hàm.
{{% /notice %}}
