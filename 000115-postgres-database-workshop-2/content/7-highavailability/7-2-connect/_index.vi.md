---
title : "Đăng ký sự kiện và theo dõi endpoint"
weight : 2
chapter : false
pre : " <b> 7.2. </b> "
---

#### Tổng quan

Trước khi failover, bạn chuẩn bị hai cách để quan sát:

- **RDS event subscription** gửi các sự kiện failover của `rdspg-lab-db` tới SNS topic `rdspg-lab-alarm-topic` đã tạo ở chương 3, để bạn nhận được email.
- Một vòng lặp trên App server in ra, mỗi 2 giây, địa chỉ IP mà endpoint phân giải tới và địa chỉ IP của server trả lời truy vấn.

#### Tạo event subscription

1. Trong Amazon RDS console:
    - (1) Chọn **Event subscriptions**.
    - (2) Chọn **Create event subscription**.

    ![Event subscriptions](/images/7/7-2/1.png)

2. Ở phần **Details** và **Target**:
    - (1) **Name**: `rdspg-lab-failover`
    - (2) **Send notifications to**: **Amazon Resource Name (ARN)**
    - (3) **ARN**: `rdspg-lab-alarm-topic`

    ![Tên và đích](/images/7/7-2/2.png)

3. Ở phần **Source**:
    - (1) **Source type**: **Instances**
    - (2) **Instances to include**: **Select specific instances**
    - (3) **Specific instances**: `rdspg-lab-db`
    - (4) **Event categories to include**: **Select specific event categories**
    - (5) **Specific event categories**: `failover`
    - (6) Chọn **Create**.

    ![Nguồn sự kiện](/images/7/7-2/3.png)

4. Subscription được tạo:
    - (1) `rdspg-lab-failover` ở trạng thái **Active**, source type **Instances**.

    ![Đã tạo event subscription](/images/7/7-2/4.png)

{{% notice note %}}
SNS topic đã có email subscription bạn xác nhận ở chương 3. Nếu dùng topic mới, hãy xác nhận email SNS gửi tới trước khi thử failover.
{{% /notice %}}

#### Theo dõi endpoint từ App server

5. Kết nối tới App server (`ssh rdspg-app`) và chạy vòng lặp sau. Để nó chạy liên tục trong cửa sổ này:

    ```bash
    while true; do
      echo "$(date +%T)  DNS=$(getent hosts $PGHOST | awk '{print $1}')  server=$(PGCONNECT_TIMEOUT=3 psql -d pglab -Atc 'SELECT inet_server_addr()' 2>&1 | head -1)"
      sleep 2
    done
    ```

    - (1) `DNS` là địa chỉ IP private mà endpoint của `rdspg-lab-db` phân giải tới.
    - (2) `server` là địa chỉ IP của DB instance trả lời, lấy từ `inet_server_addr()`. Cả hai đều là IP của primary hiện tại.

    ![Vòng lặp theo dõi](/images/7/7-2/5.png)

#### (Tuỳ chọn) AWS CLI

{{%expand "Tạo event subscription bằng AWS CLI" %}}
```bash
TOPIC=$(aws sns list-topics --query "Topics[?ends_with(TopicArn,':rdspg-lab-alarm-topic')].TopicArn" --output text)

aws rds create-event-subscription \
    --subscription-name rdspg-lab-failover \
    --sns-topic-arn $TOPIC \
    --source-type db-instance \
    --source-ids rdspg-lab-db \
    --event-categories failover \
    --enabled
```
{{% /expand %}}
