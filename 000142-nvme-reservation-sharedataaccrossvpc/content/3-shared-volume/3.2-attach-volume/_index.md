---
title : "Attach the volume to both instances"
weight : 2
chapter : false
pre : " <b> 3.2 </b> "
---

1. Select `shared-io2` and choose **Actions** → **Attach volume**. Open the **Instance** list:
   - (1) The volume is in `ap-northeast-1a`.
   - (2) The list shows both `prod-postgres` and `test-postgres`. The console only lists instances in the same Availability Zone as the volume, whatever their VPC.

   ![Instances in the same zone](/images/3-shared-volume/3.2-instance-list.png)

2. Attach the volume to the production instance:
   - (1) **Instance**: `prod-postgres`.
   - (2) **Device name**: `/dev/sdf`.
   - (3) Choose **Attach volume**.

   ![Attach to prod-postgres](/images/3-shared-volume/3.2-attach-prod.png)

3. Select `shared-io2` again and choose **Actions** → **Attach volume**:
   - (1) **Instance**: `test-postgres`.
   - (2) **Device name**: `/dev/sdf`.
   - (3) **Attached instances** already shows `prod-postgres`.
   - Choose **Attach volume**.

   ![Attach to test-postgres](/images/3-shared-volume/3.2-attach-test.png)

4. Select `shared-io2` and look at the **Details** tab:
   - (1) `io2`, 4 GiB, 100 IOPS.
   - (2) **Volume state**: **In-use**.
   - (3) **Multi-Attach enabled**: **Yes**.
   - (4) **Attached resources**: **2 instances**.

   ![Volume attached to two instances](/images/3-shared-volume/3.2-attached.png)

On Nitro instances the device name you choose is only a label. Inside the operating system the volume appears as an NVMe device, `/dev/nvme1n1` in this lab.
