---
title : "Scalability"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

#### Overview

When the load grows, you can scale an RDS for PostgreSQL database in two directions:

| Direction | How | Good for | Limit |
|---|---|---|---|
| **Vertical** (scale up) | Change the DB instance class | CPU or memory bottlenecks, write-heavy workloads | The largest instance class; a short downtime for each change |
| **Horizontal** (scale out) | Add **read replicas** | Read-heavy workloads, reports, analytics | Replicas are read-only and asynchronous, so they can lag behind the primary |

In this chapter, you create a read replica of `rdspg-lab-db`, promote it to a standalone DB instance, review vertical scaling, and work with **Multi-AZ DB clusters**: you create a Multi-AZ DB cluster from a read replica, promote it, and add a DB instance read replica to the cluster.

{{% notice warning %}}
Multi-AZ DB clusters run three DB instances of the `m`, `r`, or `c` class with local NVMe storage, which is not in the free tier. Delete them right after steps 5.4 and 5.5.
{{% /notice %}}
