---
title : "High availability"
weight : 7
chapter : false
pre : " <b> 7. </b> "
---

#### Overview

Amazon RDS for PostgreSQL offers three ways to keep a database available when a DB instance or an Availability Zone fails:

| Option | What RDS runs | Failover | Read from the copy |
|---|---|---|---|
| **Multi-AZ DB instance** | 1 primary + 1 standby in another AZ, synchronous replication | Automatic, usually 60 to 120 seconds | No |
| **Multi-AZ DB cluster** | 1 writer + 2 readable standbys in 3 AZs | Automatic, usually under 35 seconds | Yes (chapter 5.4) |
| **Read replica** | Asynchronous copy, in the same or another Region | Manual (you promote it) | Yes (chapter 5.1) |

With a **Multi-AZ DB instance**, the endpoint of the DB instance does not change. During a failover, RDS points the DNS name of the endpoint to the standby, which becomes the new primary. Your application only has to reconnect.

In this chapter, you convert `rdspg-lab-db` to a Multi-AZ DB instance, subscribe to its failover events, watch the endpoint from the App server, and force a failover with **Reboot with failover**.

{{% notice warning %}}
A Multi-AZ DB instance costs about twice as much as a Single-AZ DB instance. At the end of step 7.3, you convert `rdspg-lab-db` back to Single-AZ.
{{% /notice %}}
