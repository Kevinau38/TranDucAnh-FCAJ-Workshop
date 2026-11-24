---
title : "Cost allocation tag"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

#### Tổng quan

Nhóm theo dịch vụ hay Region cho biết bạn trả tiền cho **thứ gì**. **Cost allocation tag** cho biết chi phí đó thuộc về **ai** hay **dự án nào**: nếu tài nguyên của bạn có tag như `Project=fcj` hoặc `Env=prod`, bạn có thể nhóm và lọc Cost Explorer theo các tag này. Xem cách gắn tag cho tài nguyên ở workshop [000027](https://000027.awsfcaj.com/).

Một tag key chỉ xuất hiện trong Cost Explorer sau khi được **kích hoạt (activate)** làm cost allocation tag:

| Loại | Do ai tạo | Ví dụ |
|---|---|---|
| **User-defined** | Bạn, trên tài nguyên của mình | `Project`, `Env`, `Owner` |
| **AWS-generated** | Các dịch vụ AWS | `aws:createdBy`, `aws:cloudformation:stack-name` |

1. Ở menu bên trái của **Billing and Cost Management**, chọn **Cost Allocation Tags**:
    - (1) Các tag key tìm thấy trên tài nguyên của tài khoản. Chỉ management account của organization mới kích hoạt được.
    - (2) **Status**: tag ở trạng thái **Inactive** chưa dùng được trong Cost Explorer.
    - (3) Chọn một tag key rồi chọn **Activate**.

    ![Cost allocation tag](/images/5/5-1.png)

2. Khoảng 24 giờ sau khi kích hoạt, tag xuất hiện trong Cost Explorer: đặt **Group by** > **Dimension** là **Tag** và chọn tag key, hoặc lọc theo một giá trị tag. Chi phí của tài nguyên không có tag hiển thị là **No tag key**.

{{% notice warning %}}
Tag không áp dụng ngược cho chi phí cũ: sau khi kích hoạt, chỉ chi phí mới được gắn tag. Bạn có thể yêu cầu **Backfill tags** (tối đa 12 tháng trước) trên cùng trang.
{{% /notice %}}
