---
title : "Introduction"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---


#### Amazon CloudWatch

CloudWatch is the monitoring and observability service of AWS. The main building blocks you use in this lab are:

| Feature | What it does |
|---|---|
| **Metrics** | Time series of numbers, such as `CPUUtilization` of an EC2 instance. A metric lives in a **namespace** (for example `AWS/EC2`) and is identified by its **dimensions** (for example `InstanceId`). |
| **CloudWatch agent** | Runs on a server and sends extra metrics (memory, disk, per-process) and log files to CloudWatch. Its metrics go to the `CWAgent` namespace. |
| **Logs** | Log events are stored in **log groups** and **log streams**. **Logs Insights** queries them with a simple query language. **Metric filters** turn matching log lines into metrics. |
| **Alarms** | Watch one metric or expression and change state (`OK`, `ALARM`, `INSUFFICIENT_DATA`) when a threshold is crossed. An alarm can notify an **Amazon SNS** topic. |
| **Dashboards** | Custom pages of graphs and alarm widgets. |

#### The lab environment

The CloudFormation stack creates three instances, **FCJ-CloudWatch-Instance-A**, **B** and **C**. Each instance:

- runs a small workload that uses CPU and writes to disk in cycles (Instance A is the busiest, C the lightest), so the metrics are not flat;
- runs the CloudWatch agent, which sends `mem_used_percent`, `disk_used_percent` and per-process metrics of the agent itself to the `CWAgent` namespace, and ships `/var/log/messages` to the log group `/fcj/cloudwatch-workshop/var/log/messages`;
- has the script `/opt/fcj/logger.py`, a fake web application that writes `INFO`, `WARNING` and `ERROR` lines to syslog. You start it in section 4.2.

The instances have no inbound rules and no SSH key. You open a shell with **Session Manager**.
