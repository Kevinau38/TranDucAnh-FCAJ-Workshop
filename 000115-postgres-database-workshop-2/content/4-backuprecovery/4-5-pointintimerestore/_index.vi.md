---
title : "Khôi phục về một thời điểm"
weight : 5
chapter : false
pre : " <b> 4.5. </b> "
---

#### Tổng quan

Snapshot đưa cơ sở dữ liệu về đúng thời điểm chụp. Nhiều khi bạn cần một thời điểm chính xác hơn, ví dụ **một giây trước** khi ai đó chạy nhầm `DROP TABLE` hay `UPDATE` không có `WHERE`. Với **point-in-time recovery (PITR)**, RDS khôi phục snapshot hằng ngày gần nhất rồi áp dụng nhật ký giao dịch đến đúng giây bạn chọn.

Trong bước này, bạn giả lập một sự cố: tạo một bảng, xóa nó, rồi lấy lại bảng trên một DB instance mới được khôi phục về thời điểm nằm giữa hai lệnh đó.

{{% notice info %}}
Bạn có thể khôi phục về bất kỳ giây nào giữa earliest restorable time và **latest restorable time**. Latest restorable time thường chậm hơn hiện tại chưa đến 5 phút, vì RDS tải nhật ký giao dịch lên mỗi 5 phút.
{{% /notice %}}

#### Giả lập sự cố

1. Trên App server, tạo một bảng có 1.000 dòng:

    ```bash
    psql -d pglab -c "CREATE TABLE important_data AS SELECT g AS id, md5(g::text) AS payload, now() AS created_at FROM generate_series(1,1000) g;"
    ```

2. Đợi vài giây, rồi xóa bảng và ghi lại thời điểm xóa:

    ```bash
    psql -d pglab \
        -c "SELECT count(*), max(created_at) AS created_time FROM important_data;" \
        -c "SELECT now() AS drop_time;" \
        -c "DROP TABLE important_data;" \
        -c "SELECT count(*) FROM important_data;"
    ```

    - (1) Bảng có 1.000 dòng, tạo lúc `01:23:22` UTC.
    - (2) Lệnh xóa chạy lúc `01:23:46` UTC.
    - (3) Sau khi xóa, bảng không còn tồn tại.

    ![The accident](/images/4/4-5/1.png)

    Bạn sẽ khôi phục cơ sở dữ liệu về `01:23:35` UTC, sau khi bảng được tạo và trước khi bị xóa. Thời gian của bạn sẽ khác: hãy chọn một giây nằm giữa `created_time` và `drop_time` của bạn.

3. Đợi khoảng 5 phút, rồi kiểm tra latest restorable time trong **AWS CloudShell**:

    ```bash
    aws rds describe-db-instances \
        --db-instance-identifier rdspg-lab-db \
        --query 'DBInstances[0].{BackupRetentionPeriod:BackupRetentionPeriod,LatestRestorableTime:LatestRestorableTime}' \
        --output table
    ```

    - (1) Latest restorable time phải muộn hơn thời điểm bạn muốn khôi phục.

    ![Latest restorable time](/images/4/4-5/2.png)

#### Khôi phục về một thời điểm

4. Trong [danh sách cơ sở dữ liệu](https://console.aws.amazon.com/rds/home#databases:):
    - (1) Chọn `rdspg-lab-db`.
    - (2) Chọn **Actions**.
    - (3) Chọn **Restore to point in time**.

    ![Restore to point in time](/images/4/4-5/3.png)

5. Trong **Restore time**:
    - (1) Chọn **Custom date and time**.
    - (2) **Date**: ngày xảy ra lệnh xóa
    - (3) **Time**: thời điểm bạn chọn ở bước 2. Console dùng múi giờ máy của bạn, nên hãy đổi từ UTC. Trong ví dụ này, `01:23:35` UTC là `08:23:35` UTC+7.

    ![Restore time](/images/4/4-5/4.png)

6. Trong **Settings**:
    - (1) **Source DB instance identifier** là `rdspg-lab-db`.
    - (2) **DB instance identifier**: `rdspg-lab-db-pitr`

    ![Settings](/images/4/4-5/5.png)

7. Trong **Instance configuration** và **Availability and durability**:
    - (1) **Burstable classes**
    - (2) **Instance type**: `db.t4g.micro`
    - (3) **Single-AZ DB instance deployment (1 instance)**

    ![Instance configuration](/images/4/4-5/6.png)

8. Trong **Connectivity**:
    - (1) **DB subnet group**: `rdspg-lab-subnet-group`
    - (2) **Public access**: **No**
    - (3) **Existing VPC security groups**: chỉ `rds-sg`

    Giữ các cấu hình còn lại và chọn **Restore to point in time** ở cuối trang.

    ![Connectivity](/images/4/4-5/7.png)

9. Đợi đến khi trạng thái của `rdspg-lab-db-pitr` là **Available**, khoảng 10 đến 15 phút:
    - (1) `rdspg-lab-db-pitr` ở trạng thái **Available**.

    ![Restored DB instance](/images/4/4-5/8.png)

#### Kiểm tra dữ liệu đã khôi phục

10. Mở `rdspg-lab-db-pitr` và sao chép **Endpoint**. Trên App server, kết nối vào DB instance được khôi phục và đếm số dòng. Nhập mật khẩu master khi `psql` hỏi:

    ```bash
    psql -h <endpoint của rdspg-lab-db-pitr> -d pglab \
        -c "SELECT count(*), max(created_at) AS created_time FROM important_data;"
    ```

    - (1) Bảng `important_data` đã trở lại, đủ 1.000 dòng.

    ![Recovered data](/images/4/4-5/9.png)

Trong một sự cố thật, bước tiếp theo là chép bảng về cơ sở dữ liệu production, ví dụ bằng `pg_dump -t important_data` từ `rdspg-lab-db-pitr` rồi `psql` vào `rdspg-lab-db`, hoặc chuyển ứng dụng sang DB instance được khôi phục.

11. Xóa `rdspg-lab-db-pitr`: chọn nó, chọn **Actions** > **Delete**, bỏ chọn **Create final snapshot** và **Retain automated backups**, chọn ô xác nhận, nhập `delete me`, rồi chọn **Delete**.

#### (Tùy chọn) AWS CLI

{{%expand "Khôi phục về một thời điểm bằng AWS CLI" %}}
```bash
aws rds restore-db-instance-to-point-in-time \
    --source-db-instance-identifier rdspg-lab-db \
    --target-db-instance-identifier rdspg-lab-db-pitr \
    --restore-time 2026-11-11T01:23:35Z \
    --db-instance-class db.t4g.micro \
    --no-multi-az \
    --db-subnet-group-name rdspg-lab-subnet-group \
    --vpc-security-group-ids <ID của rds-sg> \
    --no-publicly-accessible

aws rds wait db-instance-available --db-instance-identifier rdspg-lab-db-pitr

aws rds delete-db-instance \
    --db-instance-identifier rdspg-lab-db-pitr \
    --skip-final-snapshot \
    --delete-automated-backups
```
Dùng `--use-latest-restorable-time` thay cho `--restore-time` để khôi phục về latest restorable time.
{{% /expand %}}
