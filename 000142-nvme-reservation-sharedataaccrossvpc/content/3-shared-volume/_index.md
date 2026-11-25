---
title : "Create the shared volume"
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

In this part you create one `io2` volume with Multi-Attach enabled and attach it to `prod-postgres` and `test-postgres`. The instances are in different VPCs, but in the same Availability Zone as the volume, so both can attach it.
