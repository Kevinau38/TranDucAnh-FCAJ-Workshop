---
title : "Thiết lập Multi-AZ"
weight : 1
chapter : false
pre : " <b> 7.1. </b> "
---

#### Tổng quan

Khi bạn chuyển DB instance Single-AZ sang Multi-AZ, RDS tạo snapshot của primary, khôi phục nó thành standby ở một Availability Zone khác, rồi bắt đầu replication đồng bộ. Primary vẫn hoạt động trong lúc chuyển đổi, nhưng hiệu năng có thể giảm trong khi standby bắt kịp dữ liệu.

#### Kiểm tra cấu hình hiện tại

1. Chọn **Databases** và kéo danh sách sang phải:
    - (1) `rdspg-lab-db` là DB instance `db.t4g.micro`.
    - (2) Cột **Multi-AZ** hiển thị **No**.
    - (3) Chọn `rdspg-lab-db` rồi chọn **Modify**.

    ![Multi-AZ No](/images/7/7-1/1.png)

#### Chuyển sang Multi-AZ

2. Trên trang **Modify DB instance: rdspg-lab-db**, kéo xuống **Availability & durability**:
    - (1) **Multi-AZ deployment**: **Create a standby instance (recommended for production usage)**

    ![Create a standby instance](/images/7/7-1/2.png)

3. Chọn **Continue**. Trên trang tóm tắt:
    - (1) Kiểm tra thay đổi: **Multi-AZ deployment** No → Yes.
    - (2) Chọn **Apply immediately**.
    - Đọc cảnh báo: việc chuyển đổi có thể ảnh hưởng hiệu năng. Với database production bận rộn, bạn có thể tạo read replica, chuyển replica sang Multi-AZ, rồi promote nó.
    - (3) Chọn **Modify DB instance**.

    ![Tóm tắt thay đổi](/images/7/7-1/3.png)

4. **Status** của `rdspg-lab-db` chuyển sang **Modifying**. Khi trạng thái trở lại **Available** (khoảng 10 đến 20 phút), chọn `rdspg-lab-db` và mở tab **Configuration**:
    - (1) **Multi-AZ** là **Yes**.
    - (2) **Storage operation status** có thể hiển thị **Initializing** một lúc: RDS vẫn đang nạp storage cho standby ở chế độ nền. DB instance vẫn dùng được, nhưng độ trễ có thể cao hơn cho đến khi quá trình này xong.

    ![Multi-AZ Yes](/images/7/7-1/4.png)

#### (Tuỳ chọn) AWS CLI

{{%expand "Chuyển sang Multi-AZ bằng AWS CLI" %}}
```bash
aws rds modify-db-instance \
    --db-instance-identifier rdspg-lab-db \
    --multi-az \
    --apply-immediately

aws rds wait db-instance-available --db-instance-identifier rdspg-lab-db

aws rds describe-db-instances \
    --db-instance-identifier rdspg-lab-db \
    --query 'DBInstances[0].[MultiAZ,AvailabilityZone,SecondaryAvailabilityZone]' \
    --output table
```
{{% /expand %}}
