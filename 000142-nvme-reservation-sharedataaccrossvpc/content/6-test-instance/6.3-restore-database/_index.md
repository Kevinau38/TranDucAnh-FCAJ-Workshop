---
title : "Restore the database"
weight : 3
chapter : false
pre : " <b> 6.3 </b> "
---

1. Mount the volume **read-only**, create the database `restore`, load the dump into it and query the table:

   ```bash
   sudo mkdir -p /backup && sudo mount -o ro,norecovery /dev/nvme1n1 /backup && ls -l /backup
   sudo postgresql-setup --initdb && sudo systemctl enable --now postgresql && sudo -u postgres createdb restore
   sudo -u postgres psql -q -d restore -f /backup/test_erp.sql && sudo -u postgres psql -d restore -c "SELECT * FROM clients;"
   ```

   - (1) The read-only mount shows `test_erp.sql`, written by `prod-postgres` in another VPC.
   - (2) The table `clients` in the database `restore` has the same three rows as in production.

   ![Database restored](/images/6-test-instance/6.3-restore.png)

`ro` mounts the file system read-only, and `norecovery` tells XFS not to replay its log, which would need a write. The data moved from `prod-vpc` to `test-vpc` without any network connection between the two VPCs.
