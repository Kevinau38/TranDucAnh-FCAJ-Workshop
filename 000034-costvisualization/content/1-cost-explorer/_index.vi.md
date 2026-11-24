---
title : "Làm quen Cost Explorer"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

#### Mở Cost Explorer

1. Mở [Billing and Cost Management console](https://console.aws.amazon.com/costmanagement/home#/cost-explorer) và chọn **Cost Explorer**:
    - (1) **Cost Explorer** ở menu bên trái mở một báo cáo chi phí và mức sử dụng mới.
    - (2) **Cost and usage overview**: tổng chi phí, chi phí trung bình mỗi kỳ và số dịch vụ có trong báo cáo.
    - (3) **Report parameters**: mặc định là 6 tháng gần nhất, độ chi tiết **Monthly**, nhóm theo **Service**.

    ![Cost Explorer](/images/1/1-1.png)

{{% notice info %}}
Lần đầu bật Cost Explorer trong một tài khoản, AWS chuẩn bị dữ liệu của tháng hiện tại và 12 tháng trước, có thể mất đến 24 giờ. Sau đó, Cost Explorer cập nhật dữ liệu chi phí ít nhất mỗi 24 giờ một lần.
{{% /notice %}}

2. Cuộn xuống phần **Cost and usage graph**:
    - (1) Mỗi tháng là một cột chồng: mỗi màu là một dịch vụ.
    - (2) Chú thích hiển thị các dịch vụ chi phí cao nhất; các dịch vụ nhỏ được gộp vào **Others**.

    ![Biểu đồ chi phí](/images/1/1-2.png)

3. Cuộn xuống phần **Cost and usage breakdown**:
    - (1) Chi phí của từng dịch vụ theo tháng và tổng của cả kỳ. Dòng đầu tiên là tổng của tài khoản.
    - (2) **Download as CSV** xuất bảng ra file, ví dụ để chia sẻ hoặc tự vẽ biểu đồ.

    ![Bảng chi tiết chi phí](/images/1/1-3.png)

#### Đổi độ chi tiết và kiểu biểu đồ

4. Trong **Report parameters**:
    - (1) Đặt **Granularity** là **Daily**. Dữ liệu theo ngày cho thấy các đỉnh mà góc nhìn theo tháng che mất: ở đây là một dịch vụ được tính phí mỗi tháng một lần vào cùng một ngày.
    - (2) Chuyển biểu đồ giữa **cột**, **đường** và **cột chồng**. Biểu đồ đường giúp so sánh xu hướng dễ hơn.

    ![Theo ngày, biểu đồ đường](/images/1/1-4.png)

{{% notice tip %}}
Độ chi tiết **Hourly** và dữ liệu **theo từng tài nguyên** (ví dụ chi phí của từng EC2 instance) là tính năng tuỳ chọn, bật trong **Cost Management preferences**. Chúng chỉ bao gồm 14 ngày gần nhất và tính phí theo số bản ghi sử dụng, nên chỉ bật khi thật sự cần.
{{% /notice %}}
