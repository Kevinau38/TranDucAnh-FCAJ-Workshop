---
title : "Acquire the reservation"
weight : 1
chapter : false
pre : " <b> 5.1 </b> "
---

1. On `prod-postgres`, register the key `0x1`, acquire a Write Exclusive reservation with it, and read the reservation status:

   ```bash
   sudo nvme resv-register /dev/nvme1n1 --nrkey=0x1 --rrega=0
   sudo nvme resv-acquire /dev/nvme1n1 --crkey=0x1 --rtype=1 --racqa=0
   sudo nvme resv-report /dev/nvme1n1 --eds
   ```

   | Option | Meaning |
   |---|---|
   | `--nrkey=0x1` | New reservation key of this instance |
   | `--rrega=0` | Register action `0`: register the key |
   | `--crkey=0x1` | Current key, used to prove who sends the command |
   | `--rtype=1` | Reservation type `1`: Write Exclusive |
   | `--racqa=0` | Acquire action `0`: acquire (`1` would preempt another holder) |
   | `--eds` | Report with the extended data structure, which shows the 64-bit keys |

   - (1) The key is registered.
   - (2) The reservation is acquired.
   - (3) `rtype : 1` (Write Exclusive) and `regctl : 1` (one registered instance).
   - (4) The registrant with `rkey : 1` has `rcsts : 1`, which means it holds the reservation.

   ![Write Exclusive reservation](/images/5-reserve-and-backup/5.1-acquire.png)
