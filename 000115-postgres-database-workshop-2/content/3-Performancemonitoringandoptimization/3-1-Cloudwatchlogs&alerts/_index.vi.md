---
title : "CloudWatch Logs và alarm"
weight : 1
chapter : false
pre : " <b> 3.1. </b> "
---

#### Tổng quan

Mặc định, log PostgreSQL của một RDS DB instance nằm trên chính instance và chỉ được giữ vài ngày. Khi đẩy log lên **Amazon CloudWatch Logs**, bạn có thể tìm kiếm, giữ log bao lâu tùy ý và phân tích cùng log của các ứng dụng.

Bạn cũng tạo một **CloudWatch alarm** trên metric `CPUUtilization`. Khi CPU trung bình đạt ít nhất 50% trong một phút, alarm gửi email qua một **Amazon SNS** topic.

#### Đẩy log PostgreSQL lên CloudWatch Logs

1. Trong [Amazon RDS console](https://console.aws.amazon.com/rds/), chọn `rdspg-lab-db`, rồi chọn **Modify**. Trong **Additional monitoring settings** > **Log exports**:
    - (1) Chọn **PostgreSQL log** và **Upgrade log**.

    ![Log exports](/images/3/3-1/1.png)

2. Chọn **Continue**:
    - (1) Kiểm tra thay đổi **Enable publish to CloudWatch logs**: *PostgreSQL log, Upgrade log*.
    - (2) Chọn **Apply immediately**.
    - (3) Chọn **Modify DB instance**.

    ![Summary of modifications](/images/3/3-1/2.png)

{{% notice note %}}
Việc đẩy log không khởi động lại cơ sở dữ liệu. RDS dùng service-linked role `AWSServiceRoleForRDS` để ghi vào CloudWatch Logs, nên bạn không cần tạo IAM role.
{{% /notice %}}

3. Sau vài phút, mở [CloudWatch console](https://console.aws.amazon.com/cloudwatch/):
    - (1) Chọn **Log Management**.
    - (2) Nhập `/aws/rds/instance/rdspg-lab-db` vào ô lọc.
    - (3) RDS tạo một log group cho mỗi loại log: `.../postgresql` và `.../upgrade`.

    ![Log groups](/images/3/3-1/3.png)

4. Chọn log group `.../postgresql`, rồi log stream `rdspg-lab-db`:
    - (1) Dòng *FATAL: pg_hba.conf rejects connection ... database "rdsadmin"* là lệnh `vacuumdb --all` bị lỗi ở bước 2.3.
    - (2) Các dòng *checkpoint starting* và *checkpoint complete* cho thấy PostgreSQL đang ghi các trang dữ liệu đã thay đổi xuống bộ lưu trữ.

    ![Log events](/images/3/3-1/4.png)

{{% notice tip %}}
Log group mới giữ dữ liệu vĩnh viễn (**Never expire**). Với môi trường của bạn, hãy đặt thời gian lưu cho log group (ví dụ 7 hoặc 30 ngày) để kiểm soát chi phí lưu trữ.
{{% /notice %}}

#### Tạo alarm cho mức sử dụng CPU

5. Quay lại `rdspg-lab-db` trong RDS console:
    - (1) Mở tab **Logs & events**.
    - (2) Trong **CloudWatch alarms**, chọn **Create alarm**.

    ![Create alarm](/images/3/3-1/5.png)

6. Trong **Settings**:
    - **Send notifications**: **Yes**
    - (1) **Send notifications to**: **New email or SMS topic**
    - (2) **Topic name**: `rdspg-lab-alarm-topic`
    - (3) **With these recipients**: địa chỉ email của bạn
    - (4) **Metric**: **Average** của **CPUUtilization**
    - (5) **Threshold**: **>=** `50` Percent

    ![Alarm settings](/images/3/3-1/6.png)

7. Tiếp tục trong **Settings**:
    - (1) **Evaluation period**: `1` consecutive period(s) of **1 Minute**
    - (2) Giữ **Name of alarm** mặc định: `awsrds-rdspg-lab-db-High-CPUUtilization`
    - (3) Chọn **Create alarm**.

    ![Create the alarm](/images/3/3-1/7.png)

8. Amazon SNS gửi thư *AWS Notification - Subscription Confirmation* tới email của bạn. Mở thư và chọn **Confirm subscription**. Sau đó mở [Amazon SNS console](https://console.aws.amazon.com/sns/), chọn **Topics** > `rdspg-lab-alarm-topic`:
    - (1) Subscription email ở trạng thái **Confirmed**.

    ![SNS subscription](/images/3/3-1/8.png)

{{% notice warning %}}
SNS chỉ gửi email alarm cho các subscription đã xác nhận. Nếu không thấy thư xác nhận, hãy kiểm tra thư mục spam. Liên kết trong thư hết hạn sau 3 ngày.
{{% /notice %}}

#### (Tùy chọn) AWS CLI

{{%expand "Đẩy log và tạo alarm bằng AWS CLI" %}}
```bash
# Đẩy log PostgreSQL và log nâng cấp lên CloudWatch Logs
aws rds modify-db-instance \
    --db-instance-identifier rdspg-lab-db \
    --cloudwatch-logs-export-configuration '{"EnableLogTypes":["postgresql","upgrade"]}' \
    --apply-immediately

# Tạo SNS topic và đăng ký email của bạn
TOPIC_ARN=$(aws sns create-topic --name rdspg-lab-alarm-topic --query TopicArn --output text)
aws sns subscribe --topic-arn $TOPIC_ARN --protocol email --notification-endpoint you@example.com

# Tạo alarm
aws cloudwatch put-metric-alarm \
    --alarm-name awsrds-rdspg-lab-db-High-CPUUtilization \
    --namespace AWS/RDS --metric-name CPUUtilization \
    --dimensions Name=DBInstanceIdentifier,Value=rdspg-lab-db \
    --statistic Average --period 60 --evaluation-periods 1 \
    --threshold 50 --comparison-operator GreaterThanOrEqualToThreshold \
    --alarm-actions $TOPIC_ARN
```
{{% /expand %}}
