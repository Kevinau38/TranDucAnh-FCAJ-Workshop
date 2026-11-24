---
title : "CloudWatch Logs and alarms"
weight : 1
chapter : false
pre : " <b> 3.1. </b> "
---

#### Overview

By default, the PostgreSQL log of an RDS DB instance stays on the instance and is kept for a few days. When you publish it to **Amazon CloudWatch Logs**, you can search it, keep it as long as you need, and analyze it together with the logs of your applications.

You also create a **CloudWatch alarm** on the `CPUUtilization` metric. When the average CPU is at least 50% for one minute, the alarm sends an email through an **Amazon SNS** topic.

#### Publish the PostgreSQL log to CloudWatch Logs

1. In the [Amazon RDS console](https://console.aws.amazon.com/rds/), choose `rdspg-lab-db`, then choose **Modify**. In **Additional monitoring settings** > **Log exports**:
    - (1) Select **PostgreSQL log** and **Upgrade log**.

    ![Log exports](/images/3/3-1/1.png)

2. Choose **Continue**:
    - (1) Check the change **Enable publish to CloudWatch logs**: *PostgreSQL log, Upgrade log*.
    - (2) Choose **Apply immediately**.
    - (3) Choose **Modify DB instance**.

    ![Summary of modifications](/images/3/3-1/2.png)

{{% notice note %}}
Publishing logs does not restart the database. RDS uses the service-linked role `AWSServiceRoleForRDS` to write to CloudWatch Logs, so you do not need to create an IAM role.
{{% /notice %}}

3. After a few minutes, open the [CloudWatch console](https://console.aws.amazon.com/cloudwatch/):
    - (1) Choose **Log Management**.
    - (2) Enter `/aws/rds/instance/rdspg-lab-db` in the filter box.
    - (3) RDS creates one log group for each log type: `.../postgresql` and `.../upgrade`.

    ![Log groups](/images/3/3-1/3.png)

4. Choose the `.../postgresql` log group, then the log stream `rdspg-lab-db`:
    - (1) The line *FATAL: pg_hba.conf rejects connection ... database "rdsadmin"* is the failed `vacuumdb --all` command from step 2.3.
    - (2) The *checkpoint starting* and *checkpoint complete* lines show PostgreSQL writing changed pages to storage.

    ![Log events](/images/3/3-1/4.png)

{{% notice tip %}}
New log groups keep data forever (**Never expire**). For your own environments, set a retention period on the log group (for example 7 or 30 days) to control the storage cost.
{{% /notice %}}

#### Create a CPU utilization alarm

5. Go back to `rdspg-lab-db` in the RDS console:
    - (1) Open the **Logs & events** tab.
    - (2) In **CloudWatch alarms**, choose **Create alarm**.

    ![Create alarm](/images/3/3-1/5.png)

6. In **Settings**:
    - **Send notifications**: **Yes**
    - (1) **Send notifications to**: **New email or SMS topic**
    - (2) **Topic name**: `rdspg-lab-alarm-topic`
    - (3) **With these recipients**: your email address
    - (4) **Metric**: **Average** of **CPUUtilization**
    - (5) **Threshold**: **>=** `50` Percent

    ![Alarm settings](/images/3/3-1/6.png)

7. Continue in **Settings**:
    - (1) **Evaluation period**: `1` consecutive period(s) of **1 Minute**
    - (2) Keep the default **Name of alarm**: `awsrds-rdspg-lab-db-High-CPUUtilization`
    - (3) Choose **Create alarm**.

    ![Create the alarm](/images/3/3-1/7.png)

8. Amazon SNS sends a message *AWS Notification - Subscription Confirmation* to your email address. Open it and choose **Confirm subscription**. Then open the [Amazon SNS console](https://console.aws.amazon.com/sns/), choose **Topics** > `rdspg-lab-alarm-topic`:
    - (1) The email subscription is **Confirmed**.

    ![SNS subscription](/images/3/3-1/8.png)

{{% notice warning %}}
SNS sends the alarm email only to confirmed subscriptions. If you do not see the confirmation message, check your spam folder. The link in the message expires after 3 days.
{{% /notice %}}

#### (Optional) AWS CLI

{{%expand "Publish the logs and create the alarm with the AWS CLI" %}}
```bash
# Publish the PostgreSQL and upgrade logs to CloudWatch Logs
aws rds modify-db-instance \
    --db-instance-identifier rdspg-lab-db \
    --cloudwatch-logs-export-configuration '{"EnableLogTypes":["postgresql","upgrade"]}' \
    --apply-immediately

# Create an SNS topic and subscribe your email address
TOPIC_ARN=$(aws sns create-topic --name rdspg-lab-alarm-topic --query TopicArn --output text)
aws sns subscribe --topic-arn $TOPIC_ARN --protocol email --notification-endpoint you@example.com

# Create the alarm
aws cloudwatch put-metric-alarm \
    --alarm-name awsrds-rdspg-lab-db-High-CPUUtilization \
    --namespace AWS/RDS --metric-name CPUUtilization \
    --dimensions Name=DBInstanceIdentifier,Value=rdspg-lab-db \
    --statistic Average --period 60 --evaluation-periods 1 \
    --threshold 50 --comparison-operator GreaterThanOrEqualToThreshold \
    --alarm-actions $TOPIC_ARN
```
{{% /expand %}}
