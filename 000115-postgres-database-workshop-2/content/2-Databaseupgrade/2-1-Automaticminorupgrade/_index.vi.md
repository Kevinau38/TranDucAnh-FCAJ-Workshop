---
title : "Tự động nâng cấp phiên bản phụ"
weight : 1
chapter : false
pre : " <b> 2.1. </b> "
---

#### Tổng quan

Khi bật **Auto minor version upgrade**, RDS nâng DB instance lên phiên bản phụ mới hơn trong **maintenance window**, khoảng 30 phút mỗi tuần do bạn chọn. RDS chỉ tự nâng cấp khi cả hai điều kiện sau đều đúng:

- **Auto minor version upgrade** đang bật.
- RDS đánh dấu một phiên bản phụ mới hơn của cùng phiên bản chính là đích nâng cấp tự động.

`rdspg-lab-db` được tạo với tùy chọn này tắt, nên phiên bản giữ ở 17.9 cho đến khi bạn tự nâng cấp ở bước tiếp theo.

#### Kiểm tra cấu hình hiện tại

1. Mở [Amazon RDS console](https://console.aws.amazon.com/rds/), chọn **Databases**, rồi chọn `rdspg-lab-db`:
    - (1) Mở tab **Maintenance & backups**.
    - (2) **Auto minor version upgrade** đang **Disabled**.
    - (3) **Maintenance window** là khoảng thời gian hằng tuần RDS áp dụng các bảo trì đang chờ.
    - (4) Để đổi cấu hình, chọn **Modify**. Bạn sẽ làm việc này ở bước tiếp theo cùng với việc nâng cấp phiên bản phụ.

    ![Maintenance and backups](/images/2/2-1/1.png)

{{% notice info %}}
**Pending maintenance** liệt kê các cập nhật RDS sẽ áp dụng trong maintenance window kế tiếp, ví dụ bản vá hệ điều hành hoặc một lần nâng cấp phiên bản phụ bắt buộc. Bạn có thể áp dụng ngay bằng **Apply now**.
{{% /notice %}}

#### (Tùy chọn) AWS CLI

Bạn có thể chạy các lệnh AWS CLI trong workshop này trên [AWS CloudShell](https://console.aws.amazon.com/cloudshell/). CloudShell đã có sẵn thông tin đăng nhập của console và miễn phí.

{{%expand "Kiểm tra cấu hình auto minor version upgrade" %}}
```bash
aws rds describe-db-instances \
    --db-instance-identifier rdspg-lab-db \
    --query 'DBInstances[0].[AutoMinorVersionUpgrade,PreferredMaintenanceWindow]' \
    --output table
```
{{% /expand %}}
