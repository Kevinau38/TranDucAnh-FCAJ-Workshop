---
title : "Metric filter"
weight : 3
chapter : false
pre : " <b> 4.3 </b> "
---


**Metric filter** đếm các log event khớp một mẫu và đẩy số đếm thành metric CloudWatch. Bạn dùng nó để đếm số dòng `ERROR` của `python-app`, rồi tạo alarm trên metric này ở phần 5.

1. Mở log group **/fcj/cloudwatch-workshop/var/log/messages**, chọn **Actions** (1), rồi **Create metric filter** (2).

![Create metric filter](/images/4-logs/4.3-actions.png)

2. Ở **Filter pattern** nhập mẫu dưới đây (1). Log event phải chứa cả hai từ thì mới khớp.

   ```
   "python-app" "ERROR"
   ```

![Filter pattern](/images/4-logs/4.3-pattern.png)

3. Ở **Test pattern**, chọn log stream của Instance A trong **Select log data to test** (1) rồi chọn **Test pattern** (2).

![Dữ liệu kiểm thử](/images/4-logs/4.3-test-data.png)

4. **Results** cho biết số event mẫu khớp (1) và liệt kê chúng (2). Chỉ những dòng `ERROR` của `python-app` khớp. Chọn **Next** (3).

![Kết quả kiểm thử](/images/4-logs/4.3-test-results.png)

5. Nhập **Filter name** `PythonAppErrors` (1) và **Metric namespace** `FCJ/CloudWatchWorkshop` (2).

![Tên filter và namespace](/images/4-logs/4.3-filter-name.png)

6. Nhập thông tin metric:
   - **Metric name**: `PythonAppErrorCount` (1)
   - **Metric value**: `1` (2), mỗi dòng khớp cộng thêm 1
   - **Default value**: `0` (3), để metric bằng 0 thay vì không có dữ liệu khi không có lỗi
   - **Unit**: `Count` (4)

![Thông tin metric](/images/4-logs/4.3-metric-details.png)

7. Để trống **Dimensions** và chọn **Next** (1).

![Next](/images/4-logs/4.3-dimensions-next.png)

8. Kiểm tra lại và chọn **Create metric filter** (1).

![Create metric filter](/images/4-logs/4.3-review.png)

9. Filter được tạo (1) và hiển thị trong tab **Metric filters** cùng namespace và metric (2). Sau khoảng một phút, metric xuất hiện trong **Metrics** dưới **FCJ/CloudWatchWorkshop**.

![Đã tạo metric filter](/images/4-logs/4.3-created.png)

{{% notice info %}}
Metric filter chỉ xử lý các log event đến sau khi filter được tạo, không đếm các event cũ.
{{% /notice %}}
