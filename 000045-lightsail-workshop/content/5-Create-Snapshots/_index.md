---
title : "Create snapshots"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

A **snapshot** is a point-in-time copy of an instance's disks. You can use it to restore the instance after a mistake, to create a copy of the instance, or to move to a larger plan (next chapter). Snapshots are billed for the storage they use; successive snapshots store only the data that changed.

1. Open the instance `fcj-wordpress` and choose the **Snapshots** tab.

2. Under **Manual snapshots**, choose **Create snapshot**. Enter `fcj-wordpress-snap-1` and choose **Create**.

![Create a snapshot](/images/5/1-create-snapshot.png)

3. The snapshot shows **Snapshotting...** for a few minutes. The instance keeps running while the snapshot is taken.

![Snapshot in progress](/images/5/2-snapshotting.png)

{{% notice info %}}
**Automatic snapshots** take a snapshot every day at a time you choose and keep the 7 most recent ones. Turn them on for production instances. The WordPress content is in the managed database, which Lightsail backs up automatically; the database has its own snapshots on its **Snapshots & restore** tab.
{{% /notice %}}
