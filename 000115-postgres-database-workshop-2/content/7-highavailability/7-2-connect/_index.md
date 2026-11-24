---
title : "Subscribe to events and watch the endpoint"
weight : 2
chapter : false
pre : " <b> 7.2. </b> "
---

#### Overview

Before you force a failover, prepare two ways to see it:

- An **RDS event subscription** sends the failover events of `rdspg-lab-db` to the SNS topic `rdspg-lab-alarm-topic` that you created in chapter 3, so you receive an email.
- A loop on the App server prints, every 2 seconds, the IP address that the endpoint resolves to and the IP address of the server that answers the query.

#### Create an event subscription

1. In the Amazon RDS console:
    - (1) Choose **Event subscriptions**.
    - (2) Choose **Create event subscription**.

    ![Event subscriptions](/images/7/7-2/1.png)

2. In **Details** and **Target**:
    - (1) **Name**: `rdspg-lab-failover`
    - (2) **Send notifications to**: **Amazon Resource Name (ARN)**
    - (3) **ARN**: `rdspg-lab-alarm-topic`

    ![Name and target](/images/7/7-2/2.png)

3. In **Source**:
    - (1) **Source type**: **Instances**
    - (2) **Instances to include**: **Select specific instances**
    - (3) **Specific instances**: `rdspg-lab-db`
    - (4) **Event categories to include**: **Select specific event categories**
    - (5) **Specific event categories**: `failover`
    - (6) Choose **Create**.

    ![Source](/images/7/7-2/3.png)

4. The subscription is created:
    - (1) `rdspg-lab-failover` is **Active**, with the source type **Instances**.

    ![Event subscription created](/images/7/7-2/4.png)

{{% notice note %}}
The SNS topic already has your confirmed email subscription from chapter 3. If you use a new topic, confirm the email that SNS sends you before you test the failover.
{{% /notice %}}

#### Watch the endpoint from the App server

5. Connect to the App server (`ssh rdspg-app`) and run the following loop. Leave it running in this window:

    ```bash
    while true; do
      echo "$(date +%T)  DNS=$(getent hosts $PGHOST | awk '{print $1}')  server=$(PGCONNECT_TIMEOUT=3 psql -d pglab -Atc 'SELECT inet_server_addr()' 2>&1 | head -1)"
      sleep 2
    done
    ```

    - (1) `DNS` is the private IP address that the endpoint of `rdspg-lab-db` resolves to.
    - (2) `server` is the IP address of the DB instance that answers, returned by `inet_server_addr()`. Both are the IP address of the current primary.

    ![Watch loop](/images/7/7-2/5.png)

#### (Optional) AWS CLI

{{%expand "Create the event subscription with the AWS CLI" %}}
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
