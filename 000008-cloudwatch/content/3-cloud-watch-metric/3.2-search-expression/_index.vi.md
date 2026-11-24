---
title : "Search expression"
weight : 2
chapter : false
pre : " <b> 3.2 </b> "
---


Chọn từng metric một không hiệu quả khi số lượng lớn. **Search expression** tìm mọi metric khớp truy vấn và vẽ chúng cùng nhau. Instance mới khớp truy vấn sẽ tự động xuất hiện.

1. Ở tab **Graphed metrics** chọn **Clear graph**. Ở tab **Browse**, chọn **All**, rồi **CWAgent**, rồi **ImageId, InstanceId, InstanceType** (1). Chọn **Graph search** (2).

![Dimension của CWAgent](/images/3-metrics/3.2-cwagent-dims.png)

2. Ở tab **Graphed metrics** xuất hiện expression `e1` với lệnh `SEARCH(...)` trong cột **Details** (1). Biểu đồ hiển thị mức dùng bộ nhớ của cả ba instance (2).

![Graph search](/images/3-metrics/3.2-graph-search.png)

3. Chọn biểu tượng bút chì cạnh expression, thay bằng expression dưới đây (1) rồi chọn **Apply** (2).

   ```
   SEARCH('Namespace="CWAgent" MetricName="disk_used_percent"', 'Average', 300)
   ```

![Sửa search expression](/images/3-metrics/3.2-edit-disk.png)

   Biểu đồ giờ hiển thị mức dùng ổ đĩa gốc của ba instance (2), tìm theo namespace và tên metric (1).

![disk_used_percent](/images/3-metrics/3.2-disk-result.png)

4. Search expression cũng nhận từ khoá tự do. Đổi expression thành dòng dưới đây (1) rồi chọn **Apply** (2).

   ```
   SEARCH('Namespace="CWAgent" used', 'Average', 300)
   ```

![Tìm theo từ khoá](/images/3-metrics/3.2-edit-used.png)

   Mọi metric trong `CWAgent` có tên chứa từ `used` đều được vẽ: `mem_used_percent` và `disk_used_percent` (1).

![Kết quả tìm theo từ khoá](/images/3-metrics/3.2-used-result.png)
