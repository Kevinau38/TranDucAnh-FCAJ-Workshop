---
title : "Nâng cấp read replica thành DB instance độc lập"
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### Tổng quan

**Promote** một read replica sẽ dừng việc đồng bộ và biến replica thành một DB instance độc lập bình thường, nhận được lệnh ghi. Bạn dùng nó, ví dụ:

- để khôi phục khi mất primary của một triển khai Single-AZ,
- để chuyển cơ sở dữ liệu sang Region khác (promote một replica khác Region),
- để tách cơ sở dữ liệu, hoặc tạo bản sao thử nghiệm với dữ liệu gần nhất.

{{% notice warning %}}
Không thể hoàn tác việc promote. DB instance đã promote không còn nhận thay đổi từ primary cũ. Trước khi promote, hãy dừng ghi vào primary và đợi replica lag gần bằng 0, nếu không các giao dịch cuối sẽ bị mất.
{{% /notice %}}

#### Promote replica

1. Chọn `rdspg-lab-db-read`:
    - (1) Chọn replica trong **Related**.
    - (2) Chọn **Actions**.
    - (3) Chọn **Promote**.

    ![Promote](/images/5/5-2/1.png)

2. Trên trang **Promote read replica**:
    - (1) Giữ **Enable automated backup**. Replica không có bản sao lưu riêng; sau khi promote, nó cần sao lưu như mọi DB instance khác.
    - (2) **Backup retention period**: `1` ngày
    - (3) Chọn **Promote read replica**.

    ![Promote settings](/images/5/5-2/2.png)

3. Cả hai DB instance chuyển sang trạng thái **Modifying** trong vài phút. RDS khởi động lại replica ở chế độ đọc-ghi và chụp bản sao lưu đầu tiên:
    - (1) `rdspg-lab-db` và `rdspg-lab-db-read` giờ là hai DB instance riêng biệt với vai trò **Instance**. Instance vừa promote ở trạng thái **Backing-up** thêm vài phút.

    ![Standalone instances](/images/5/5-2/3.png)

4. Bạn không cần `rdspg-lab-db-read` nữa. Chọn nó, chọn **Actions** > **Delete**, bỏ chọn **Create final snapshot** và **Retain automated backups**, chọn ô xác nhận, nhập `delete me`, rồi chọn **Delete**.

#### (Tùy chọn) AWS CLI

{{%expand "Promote và xóa bằng AWS CLI" %}}
```bash
aws rds promote-read-replica \
    --db-instance-identifier rdspg-lab-db-read \
    --backup-retention-period 1

aws rds wait db-instance-available --db-instance-identifier rdspg-lab-db-read

aws rds delete-db-instance \
    --db-instance-identifier rdspg-lab-db-read \
    --skip-final-snapshot \
    --delete-automated-backups
```
{{% /expand %}}
