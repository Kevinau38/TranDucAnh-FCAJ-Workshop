---
title : "Force a failover"
weight : 3
chapter : false
pre : " <b> 7.3. </b> "
---

#### Overview

**Reboot with failover** simulates the loss of the primary: RDS restarts the DB instance and promotes the standby in the other Availability Zone to primary. The endpoint stays the same, but its DNS record now points to the new primary. Keep the loop from step 7.2 running while you do this step.

#### Reboot with failover

1. In the list of databases:
    - (1) Select `rdspg-lab-db`.
    - (2) Choose **Actions**.
    - (3) Choose **Reboot**.

    ![Reboot](/images/7/7-3/1.png)

2. On the **Reboot DB Instance** page:
    - (1) Check that the DB instance is `rdspg-lab-db`.
    - (2) Select **Reboot With Failover?**.
    - (3) Choose **Confirm**.

    ![Reboot with failover](/images/7/7-3/2.png)

    The **Status** changes to **Rebooting**, then **Available** after about one minute.

#### Check the events

3. Choose `rdspg-lab-db` and open the **Logs & events** tab:
    - (1) In **Recent events**, read from the bottom up: *Multi-AZ instance failover started*, *DB instance restarted*, *Multi-AZ instance failover completed*, and *The user requested a failover of the DB instance*.

    ![Failover events](/images/7/7-3/3.png)

    Because of the event subscription `rdspg-lab-failover`, you also receive these events by email from Amazon SNS.

#### Check the endpoint from the App server

4. Go back to the App server window:
    - (1) Before the failover, `DNS` and `server` show the IP address of the old primary.
    - In the framed lines, the connection fails for about 20 seconds: first a timeout on the old primary, then *Connection refused* while the new primary starts. The endpoint already resolves to the new IP address.
    - (2) After the failover, the same endpoint resolves to the new IP address, the address of the new primary in the other Availability Zone, and the queries succeed again.

    ![IP address after failover](/images/7/7-3/4.png)

    Press **Ctrl+C** to stop the loop.

{{% notice tip %}}
Applications that keep connections open must reconnect after a failover. Use a short DNS cache time (for example, a JVM DNS TTL of 30 seconds or less), retry failed connections, and consider [Amazon RDS Proxy](https://aws.amazon.com/rds/proxy/), which keeps client connections open during a failover.
{{% /notice %}}

#### Convert back to Single-AZ

5. To stop paying for the standby, convert `rdspg-lab-db` back to Single-AZ. In AWS CloudShell, run:

    ```bash
    aws rds modify-db-instance \
        --db-instance-identifier rdspg-lab-db \
        --no-multi-az --apply-immediately \
        --query 'DBInstance.[DBInstanceIdentifier,MultiAZ,PendingModifiedValues.MultiAZ]' \
        --output text
    ```

    The output `rdspg-lab-db  True  False` means that the instance is still Multi-AZ and that the change to Single-AZ is pending. RDS removes the standby within a few minutes, without a restart. You can also do this in the console with **Modify** > **Do not create a standby instance**.

#### (Optional) AWS CLI

{{%expand "Force a failover with the AWS CLI" %}}
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
