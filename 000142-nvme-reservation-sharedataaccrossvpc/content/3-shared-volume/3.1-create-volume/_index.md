---
title : "Create the io2 volume"
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---

1. In the EC2 console, open **Volumes** and choose **Create volume**.
   - (1) **Volume type**: **Provisioned IOPS SSD (io2)**.
   - (2) **Size**: `4` GiB, the minimum for `io2`.
   - (3) **IOPS**: `100`, the minimum. The lab only moves a few kilobytes.
   - (4) **Availability Zone**: `ap-northeast-1a`, the zone of the two subnets.

   ![Volume settings](/images/3-shared-volume/3.1-volume-settings.png)

2. Scroll down:
   - (1) Select **Enable Multi-Attach**.
   - (2) Choose **Add new tag** and enter **Key** `Name`, **Value** `shared-io2`.
   - Choose **Create volume**.

   ![Multi-Attach and tag](/images/3-shared-volume/3.1-multi-attach.png)

The console shows a note about Windows Server Failover Clustering. It does not apply to Linux instances.
