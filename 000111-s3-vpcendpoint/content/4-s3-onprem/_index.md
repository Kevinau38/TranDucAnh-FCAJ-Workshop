---
title : "Access S3 from on-premises"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

Gateway endpoints only work for traffic that originates **inside** the VPC. To reach S3 privately from a data center you need an **Interface endpoint**, whose private IPs are reachable over the hybrid link.
