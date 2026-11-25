---
title : "Restore on the test instance"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---

In this part you work on `test-postgres`. You register a second key, check that the volume blocks writes from this instance, and restore the backup from a read-only mount.

Open a Session Manager session on `test-postgres`, run `bash -l` and `cd ~`, then install the packages as in step 4.1:

```bash
sudo dnf install -y -q nvme-cli postgresql18-server
```
