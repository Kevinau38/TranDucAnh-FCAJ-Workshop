---
title : "Hand over write access"
weight : 7
chapter : false
pre : " <b> 7. </b> "
---

Suppose the test environment now needs to write to the volume. The production instance releases its reservation, and the test instance acquires it.

1. On `prod-postgres`, release the reservation:

   ```bash
   sudo nvme resv-release /dev/nvme1n1 --crkey=0x1 --rtype=1 --rrela=0
   sudo nvme resv-report /dev/nvme1n1 --eds
   ```

   - (1) The reservation is released (`--rrela=0` means release; `1` would clear all registrations).
   - (2) `rtype : 0`: no instance holds a reservation.
   - (3) `rkey : 1` now has `rcsts : 0`. Both keys stay registered.

   ![Reservation released](/images/7-switch-writer/7.1-release.png)

2. On `test-postgres`, acquire the reservation with the key `0x2`, remount the volume read-write and write a file:

   ```bash
   sudo nvme resv-acquire /dev/nvme1n1 --crkey=0x2 --rtype=1 --racqa=0
   sudo umount /backup && sudo mount /dev/nvme1n1 /backup && echo "written by test-postgres at $(date -u)" | sudo tee /backup/handover.txt && ls -l /backup
   sudo nvme resv-report /dev/nvme1n1 --eds | grep -E 'rtype|rcsts|rkey'
   ```

   - (1) `test-postgres` acquires the Write Exclusive reservation.
   - (2) The write succeeds.
   - (3) `handover.txt` sits next to the backup file.
   - (4) `rkey : 2` now has `rcsts : 1`: `test-postgres` is the only instance that can write.

   ![Test instance holds the reservation](/images/7-switch-writer/7.2-acquire.png)

Applications that share a volume use the same pattern. A cluster manager such as Pacemaker registers a key on each node and lets only the active node hold the reservation. If the active node stops responding, another node **preempts** the reservation (`resv-acquire --racqa=1 --prkey=<old key>`), and the volume immediately blocks any late write from the old node.
