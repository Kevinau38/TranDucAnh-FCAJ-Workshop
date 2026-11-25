---
title : "Back up the database to the shared volume"
weight : 2
chapter : false
pre : " <b> 5.2 </b> "
---

1. Create an XFS file system on the shared volume and a mount point. Do this only once, on the instance that holds the reservation:

   ```bash
   sudo mkfs.xfs -q /dev/nvme1n1
   sudo mkdir -p /backup
   ```

2. Mount the volume, write a dump of `test_erp` with `pg_dump`, then unmount the volume:

   ```bash
   sudo mount /dev/nvme1n1 /backup && df -h /backup
   sudo chown postgres:postgres /backup
   sudo -u postgres pg_dump -d test_erp -f /backup/test_erp.sql && ls -l /backup && grep -A 4 '^COPY' /backup/test_erp.sql
   sudo umount /backup && echo UNMOUNTED
   ```

   - (1) The 4 GiB volume is mounted at `/backup`.
   - (2) The dump file `test_erp.sql` is written to the volume.
   - (3) The dump contains the three rows of `clients`.
   - (4) The volume is unmounted.

   ![Backup on the shared volume](/images/5-reserve-and-backup/5.2-backup.png)

Unmounting is important. XFS keeps data and metadata in the page cache of the instance, and a second instance cannot see them until they are written to the disk. `umount` flushes everything to the volume, so the test instance reads a complete, consistent file system. `prod-postgres` still holds the reservation after the unmount.
