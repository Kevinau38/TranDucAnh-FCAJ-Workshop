---
title : "Khôi phục snapshot"
weight : 4
chapter : false
pre : " <b> 4.4. </b> "
---

#### Tổng quan

Bạn khôi phục manual snapshot ở bước 4.3 thành một DB instance **mới**, `rdspg-lab-db-restored`. DB instance được khôi phục có dữ liệu tại thời điểm chụp snapshot. `rdspg-lab-db` không bị thay đổi và vẫn chạy.

{{% notice warning %}}
Trang khôi phục không chép mọi cấu hình từ nguồn. Mặc định, nó có thể chọn triển khai **Multi-AZ**, instance class **Standard** và security group **default**. Hãy kiểm tra từng cấu hình bên dưới, nếu không DB instance được khôi phục có thể tốn nhiều chi phí hơn và App server không kết nối tới được.
{{% /notice %}}

#### Khôi phục snapshot

1. Trong RDS console, chọn **Snapshots**, rồi tab **Manual**. Nhập `rdspg` vào ô lọc:
    - (1) Chọn `rdspg-lab-db-manual-snapshot`.
    - (2) Chọn **Actions**.
    - (3) Chọn **Restore snapshot**.

    ![Restore snapshot](/images/4/4-4/1.png)

2. Trong **Availability and durability**, chọn **Single-AZ DB instance deployment (1 instance)**. Trong **Settings**:
    - (1) **DB snapshot ID** là `rdspg-lab-db-manual-snapshot`.
    - (2) **DB instance identifier**: `rdspg-lab-db-restored`
    - (3) **DB instance class**: **Burstable classes**, rồi chọn `db.t4g.micro` trong **Instance type**.

    ![Restore settings](/images/4/4-4/2.png)

3. Trong **Connectivity**, giữ `rdspg-lab-vpc` và:
    - (1) **DB subnet group**: `rdspg-lab-subnet-group`
    - (2) **Public access**: **No**
    - (3) **Existing VPC security groups**: bỏ `default` và chỉ chọn `rds-sg`.

    ![Connectivity](/images/4/4-4/3.png)

4. Giữ các cấu hình còn lại, rồi chọn **Restore DB instance** ở cuối trang:
    - (1) RDS hiện thông báo *Snapshot is being restored*.
    - (2) `rdspg-lab-db-restored` xuất hiện trong danh sách với trạng thái **Creating**.

    ![Restoring](/images/4/4-4/4.png)

5. Sau khoảng 10 phút:
    - (1) `rdspg-lab-db-restored` ở trạng thái **Available**, cùng engine và instance class với `rdspg-lab-db`.

    ![Restored DB instance](/images/4/4-4/5.png)

{{% notice tip %}}
DB instance được khôi phục có endpoint riêng. Để kiểm tra dữ liệu, hãy kết nối từ App server bằng `psql -h <endpoint của rdspg-lab-db-restored> -d pglab`. Mật khẩu master giống `rdspg-lab-db`.
{{% /notice %}}

6. Bạn không cần `rdspg-lab-db-restored` nữa. Chọn nó, chọn **Actions** > **Delete**, bỏ chọn **Create final snapshot** và **Retain automated backups**, chọn ô xác nhận, nhập `delete me`, rồi chọn **Delete**.

#### (Tùy chọn) AWS CLI

{{%expand "Khôi phục và xóa bằng AWS CLI" %}}
```bash
aws rds restore-db-instance-from-db-snapshot \
    --db-snapshot-identifier rdspg-lab-db-manual-snapshot \
    --db-instance-identifier rdspg-lab-db-restored \
    --db-instance-class db.t4g.micro \
    --no-multi-az \
    --db-subnet-group-name rdspg-lab-subnet-group \
    --vpc-security-group-ids <ID của rds-sg> \
    --no-publicly-accessible

aws rds wait db-instance-available --db-instance-identifier rdspg-lab-db-restored

aws rds delete-db-instance \
    --db-instance-identifier rdspg-lab-db-restored \
    --skip-final-snapshot \
    --delete-automated-backups
```
{{% /expand %}}
