---
title : "Check that writes are blocked"
weight : 2
chapter : false
pre : " <b> 6.2 </b> "
---

1. Try to write one 4 KiB block directly to the device, near the end of the volume, then read 1 MiB from the start of the volume:

   ```bash
   sudo dd if=/dev/zero of=/dev/nvme1n1 bs=4k count=1 seek=1048000 oflag=direct
   sudo dd if=/dev/nvme1n1 of=/dev/null bs=4k count=256 iflag=direct
   ```

   - (1) The write fails with `Invalid exchange`. This is how Linux reports the NVMe status **Reservation Conflict**: the volume rejected the write because `prod-postgres` holds a Write Exclusive reservation.
   - (2) The read succeeds. A Write Exclusive reservation still lets the other instances read.

   ![Write blocked, read allowed](/images/6-test-instance/6.2-write-blocked.png)

{{% notice warning %}}
Never write to a raw device that holds a file system. The write test targets a block near the end of the 4 GiB volume, which the small XFS file system does not use, and it is rejected anyway. Without the reservation, the same command would succeed.
{{% /notice %}}
