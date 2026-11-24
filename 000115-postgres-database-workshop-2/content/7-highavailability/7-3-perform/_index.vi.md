---
title : "Chủ động failover"
weight : 3
chapter : false
pre : " <b> 7.3. </b> "
---

#### Tổng quan

**Reboot with failover** mô phỏng việc mất primary: RDS khởi động lại DB instance và nâng standby ở Availability Zone còn lại thành primary. Endpoint giữ nguyên, nhưng bản ghi DNS của nó giờ trỏ tới primary mới. Hãy để vòng lặp ở bước 7.2 tiếp tục chạy trong lúc bạn làm bước này.

#### Reboot with failover

1. Trong danh sách database:
    - (1) Chọn `rdspg-lab-db`.
    - (2) Chọn **Actions**.
    - (3) Chọn **Reboot**.

    ![Reboot](/images/7/7-3/1.png)

2. Trên trang **Reboot DB Instance**:
    - (1) Kiểm tra DB instance là `rdspg-lab-db`.
    - (2) Chọn **Reboot With Failover?**.
    - (3) Chọn **Confirm**.

    ![Reboot with failover](/images/7/7-3/2.png)

    **Status** chuyển sang **Rebooting**, rồi **Available** sau khoảng một phút.

#### Kiểm tra sự kiện

3. Chọn `rdspg-lab-db` và mở tab **Logs & events**:
    - (1) Trong **Recent events**, đọc từ dưới lên: *Multi-AZ instance failover started*, *DB instance restarted*, *Multi-AZ instance failover completed*, và *The user requested a failover of the DB instance*.

    ![Sự kiện failover](/images/7/7-3/3.png)

    Nhờ event subscription `rdspg-lab-failover`, bạn cũng nhận được các sự kiện này qua email từ Amazon SNS.

#### Kiểm tra endpoint từ App server

4. Quay lại cửa sổ App server:
    - (1) Trước failover, `DNS` và `server` hiển thị IP của primary cũ.
    - Ở các dòng trong khung, kết nối lỗi khoảng 20 giây: đầu tiên là timeout tới primary cũ, sau đó là *Connection refused* trong lúc primary mới khởi động. Lúc này endpoint đã phân giải sang IP mới.
    - (2) Sau failover, cùng endpoint đó phân giải tới IP mới, là IP của primary mới ở Availability Zone còn lại, và truy vấn chạy lại bình thường.

    ![IP sau failover](/images/7/7-3/4.png)

    Nhấn **Ctrl+C** để dừng vòng lặp.

{{% notice tip %}}
Ứng dụng giữ kết nối lâu dài phải kết nối lại sau failover. Hãy dùng thời gian cache DNS ngắn (ví dụ DNS TTL của JVM từ 30 giây trở xuống), thử lại khi kết nối lỗi, và cân nhắc [Amazon RDS Proxy](https://aws.amazon.com/rds/proxy/), dịch vụ giữ kết nối của client trong lúc failover.
{{% /notice %}}

#### Chuyển về Single-AZ

5. Để không phải trả tiền cho standby, chuyển `rdspg-lab-db` về Single-AZ. Trong AWS CloudShell, chạy:

    ```bash
    aws rds modify-db-instance \
        --db-instance-identifier rdspg-lab-db \
        --no-multi-az --apply-immediately \
        --query 'DBInstance.[DBInstanceIdentifier,MultiAZ,PendingModifiedValues.MultiAZ]' \
        --output text
    ```

    Kết quả `rdspg-lab-db  True  False` nghĩa là instance vẫn đang Multi-AZ và thay đổi sang Single-AZ đang chờ áp dụng. RDS gỡ standby trong vài phút, không cần khởi động lại. Bạn cũng có thể làm trong console bằng **Modify** > **Do not create a standby instance**.

#### (Tuỳ chọn) AWS CLI

{{%expand "Chủ động failover bằng AWS CLI" %}}
```bash
aws rds reboot-db-instance \
    --db-instance-identifier rdspg-lab-db \
    --force-failover

aws rds describe-events \
    --source-type db-instance --source-identifier rdspg-lab-db \
    --event-categories failover --duration 60 \
    --query 'Events[].[Date,Message]' --output table
```
{{% /expand %}}
