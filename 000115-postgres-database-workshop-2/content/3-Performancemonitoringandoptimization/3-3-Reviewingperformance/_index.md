---
title : "Review performance"
weight : 3
chapter : false
pre : " <b> 3.3. </b> "
---

#### Overview

**Database load** is measured in **average active sessions (AAS)**: the average number of sessions that are running or waiting at the same time. Database Insights splits the load by **wait event**, so you can see what the sessions spend their time on:

| Wait event | Meaning |
|---|---|
| `CPU` | The session is running on the CPU or waiting for a CPU |
| `IO:DataFileRead` | The session reads a page that is not in memory (shared buffers) |
| `IO:WalSync`, `LWLock:WALWrite` | The session waits for the transaction log (WAL) to be written at `COMMIT` |
| `Lock:transactionid`, `Lock:tuple` | The session waits for a row that another transaction is updating |

When the load is higher than the number of vCPUs (the **Max vCPUs** line), sessions wait for each other.

#### Review the database load

1. Open the [CloudWatch console](https://console.aws.amazon.com/cloudwatch/), choose **Database Insights**, then choose **Database Instance** and `rdspg-lab-db`. Choose the **30m** time range:
    - (1) During the load test, the database load reaches about 45 active sessions, and almost all of it is green.
    - (2) Green is the `CPU` wait event. The dashed **Max vCPUs** line is at 2, the number of vCPUs of `db.t4g.micro`.

    ![Database load](/images/3/3-3/1.png)

2. Scroll down to the **DB Load Analysis** tab:
    - (1) Open **Top SQL**.
    - (2) The statement with the highest load is `UPDATE pgbench_accounts ...`, the update of the biggest table. Most of its load is also `CPU`.

    ![Top SQL](/images/3/3-3/2.png)

{{% notice tip %}}
Use **Top waits**, **Top hosts**, **Top users** and **Top databases** to slice the same load by other dimensions. For example, **Top hosts** shows that all sessions come from the private IP address of the App server.
{{% /notice %}}

#### Check the alarm

3. Scroll up to **Alarms**:
    - (1) `awsrds-rdspg-lab-db-High-CPUUtilization` is in the **In alarm** state: CPU utilization reached about 95%, above the 50% threshold.

    ![Alarm in Database Insights](/images/3/3-3/3.png)

Check your mailbox. Amazon SNS sends the message *ALARM: "awsrds-rdspg-lab-db-High-CPUUtilization"* within a minute or two of the state change.

#### Conclusion

The database spends almost all its time on the CPU, and there are many more active sessions than vCPUs. The bottleneck is CPU, not storage I/O or locks. In the next step you give the database more vCPUs and memory.
