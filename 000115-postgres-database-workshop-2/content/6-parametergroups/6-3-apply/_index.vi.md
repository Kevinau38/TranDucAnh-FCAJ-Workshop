---
title : "Áp dụng parameter group"
weight : 3
chapter : false
pre : " <b> 6.3. </b> "
---

#### Tổng quan

Mỗi DB instance chỉ dùng một DB parameter group tại một thời điểm. Khi bạn đổi group của DB instance, RDS đánh dấu **Pending reboot**: group mới, kể cả các tham số dynamic, chỉ có hiệu lực sau khi bạn reboot DB instance. Sau đó, thay đổi các tham số dynamic trong cùng group sẽ được áp dụng mà không cần reboot.

#### Gắn parameter group vào rdspg-lab-db

1. Chọn **Databases**, chọn `rdspg-lab-db`, rồi chọn **Modify**. Kéo xuống **Additional configuration** > **Database options**:
    - (1) **DB parameter group**: `rdspg-lab-pg18`

    ![DB parameter group](/images/6/6-3/1.png)

2. Chọn **Continue**. Trên trang tóm tắt:
    - (1) Kiểm tra thay đổi: **DB parameter group** `default.postgres18` → `rdspg-lab-pg18`.
    - (2) Chọn **Apply immediately**.
    - (3) Chọn **Modify DB instance**.

    ![Tóm tắt thay đổi](/images/6/6-3/2.png)

3. Chọn `rdspg-lab-db` và mở tab **Configuration**:
    - (1) **DB instance parameter group** là `rdspg-lab-pg18` với trạng thái **Pending reboot**.

    ![Pending reboot](/images/6/6-3/3.png)

#### Reboot DB instance

4. Chọn **Actions** > **Reboot**:
    - (1) Kiểm tra DB instance là `rdspg-lab-db`.
    - (2) Chọn **Confirm**. Database không truy cập được trong khoảng một phút.

    ![Reboot](/images/6/6-3/4.png)

    Khi **Status** trở lại **Available**, trạng thái của parameter group là **In sync**.

#### Kiểm tra giá trị mới

5. Từ App server (`ssh rdspg-app`), chạy:

    ```bash
    psql -d pglab -c "SHOW log_connections;" \
                  -c "SHOW log_min_duration_statement;" \
                  -c "SELECT pg_sleep(5);"
    ```

    - (1) `log_connections` là `receipt,authentication,authorization`.
    - (2) `log_min_duration_statement` là `4s` (4000 ms).
    - (3) `pg_sleep(5)` chạy 5 giây, lâu hơn ngưỡng 4 giây, nên PostgreSQL ghi câu lệnh này vào log.

    ![SHOW](/images/6/6-3/5.png)

{{% notice info %}}
Log của PostgreSQL giờ có các dòng như `connection authorized: user=postgres database=pglab` và `duration: 5005.123 ms  statement: SELECT pg_sleep(5);`. Xem chúng trong console (**Logs & events** > **Logs**) hoặc, nếu bạn export log PostgreSQL, trong CloudWatch Logs (`/aws/rds/instance/rdspg-lab-db/postgresql`).
{{% /notice %}}

{{% notice warning %}}
`log_connections` và `log_min_duration_statement` thấp có thể sinh rất nhiều log trên database bận rộn. Trong môi trường production, hãy chọn ngưỡng chỉ bắt những câu truy vấn bạn cần tối ưu.
{{% /notice %}}

#### (Tuỳ chọn) AWS CLI

{{%expand "Gắn parameter group và reboot bằng AWS CLI" %}}
```bash
aws rds modify-db-instance \
    --db-instance-identifier rdspg-lab-db \
    --db-parameter-group-name rdspg-lab-pg18 \
    --apply-immediately

aws rds reboot-db-instance --db-instance-identifier rdspg-lab-db

aws rds wait db-instance-available --db-instance-identifier rdspg-lab-db

aws rds describe-db-instances \
    --db-instance-identifier rdspg-lab-db \
    --query 'DBInstances[0].DBParameterGroups' --output table
```
{{% /expand %}}
