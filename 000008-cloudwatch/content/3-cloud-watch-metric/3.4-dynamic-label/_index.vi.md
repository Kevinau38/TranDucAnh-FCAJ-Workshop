---
title : "Dynamic label"
weight : 4
chapter : false
pre : " <b> 3.4 </b> "
---


Chú thích của search expression thường chứa tên dài, khó đọc. **Dynamic label** tạo nhãn cho từng chuỗi từ các thuộc tính của nó và tự cập nhật.

1. Chọn **Clear graph**. Ở tab **Browse**, chọn **CWAgent**, rồi **ImageId, InstanceId, InstanceType, exe, process_name** (1). Đây là các metric theo process (`procstat`) của chính process CloudWatch agent.

![Dimension procstat](/images/3-metrics/3.4-cwagent-dims.png)

2. Tìm `memory_rss` (1) rồi chọn **Graph search** (2). Có ba dòng khớp, mỗi instance một dòng (3).

![procstat_memory_rss](/images/3-metrics/3.4-memory-rss.png)

3. Ở tab **Graphed metrics**, chọn **Add dynamic label**, **Properties** (1), rồi **Dim.DimName - dimension value** (2).

![Add dynamic label](/images/3-metrics/3.4-dynamic-label-menu.png)

4. Nhãn trở thành `[dimension: ${PROP('Dim.DimName')}]` (1). `DimName` chỉ là chỗ giữ chỗ nên chú thích hiển thị `--` (2).

![Nhãn giữ chỗ](/images/3-metrics/3.4-dimname-label.png)

5. Chọn biểu tượng bút chì ở cột **Label**, thay nhãn bằng dòng dưới đây (1) rồi chọn **Apply** (2).

   ```
   ${PROP('Dim.process_name')} - ${PROP('Dim.InstanceId')} - ${PROP('MetricName')}
   ```

![Sửa nhãn](/images/3-metrics/3.4-edit-label.png)

6. Mỗi chuỗi giờ có nhãn rõ ràng: tên process, instance và tên metric (1).

![Dynamic label](/images/3-metrics/3.4-label-result.png)
