---
title : "Register the test key"
weight : 1
chapter : false
pre : " <b> 6.1 </b> "
---

1. On `test-postgres`, register the key `0x2` and read the reservation status:

   ```bash
   sudo nvme resv-register /dev/nvme1n1 --nrkey=0x2 --rrega=0
   sudo nvme resv-report /dev/nvme1n1 --eds
   ```

   - (1) The key `0x2` is registered.
   - (2) The reservation is still type `1` (Write Exclusive), and `regctl : 2`: two instances are registered.
   - (3) `rkey : 1` (`prod-postgres`) has `rcsts : 1`: it still holds the reservation.
   - (4) `rkey : 2` (`test-postgres`) has `rcsts : 0`: it is registered but does not hold the reservation.

   ![Two registered keys](/images/6-test-instance/6.1-register.png)

Both instances see the same status, because the reservation is stored on the volume, not on the instances.
