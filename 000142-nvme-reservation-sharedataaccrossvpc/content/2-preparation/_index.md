---
title : "Preparation"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

In this part you build the network and the two EC2 instances:

| Step | What you create |
|---|---|
| 2.1, 2.2 | `prod-vpc` (10.10.0.0/16) and `test-vpc` (10.20.0.0/16), each with one private subnet in the same Availability Zone |
| 2.3 | Security groups, and interface endpoints for Session Manager in both VPCs |
| 2.4 | The IAM role `FCJ-NVMe-EC2-Role` for the instances |
| 2.5 | The instances `prod-postgres` and `test-postgres` |

The two VPCs are not connected to each other: there is no peering, no transit gateway and no VPN between them.
