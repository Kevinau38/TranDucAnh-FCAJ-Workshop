---
title : "Migrate to a larger instance"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---

When the blog gets more traffic, you can move it to a plan with more memory. A Lightsail instance cannot change its plan in place: you create a new, larger instance from a snapshot, then move the static IP to it. Because visitors use the static IP, the address of the site does not change.

1. On the instance page, point to **Change plan** next to the instance size. Lightsail explains the method: create a snapshot, then create an instance from that snapshot.

![Change plan](/images/6/1-change-plan.png)

2. In the left menu, choose **Snapshots**. Expand the snapshots of `fcj-wordpress`. When `fcj-wordpress-snap-1` is complete, open its actions menu (⋮) and choose **Create new instance**.

![Snapshot actions](/images/6/2-snapshot-menu.png)

3. The page shows the snapshot source and the same Availability Zone.

![Create from snapshot](/images/6/3-from-snapshot.png)

4. Under **Select a size**, choose the **USD 12** plan (2 GB memory, 60 GB SSD). You can choose the same size or larger than the snapshot, but not smaller.

![Larger plan](/images/6/4-plan.png)

5. Under **Identify your instance**, enter `fcj-wordpress-2gb`. The tag `Project` = `fcj-lightsail` is copied from the snapshot. Choose **Create instance**.

![Name the new instance](/images/6/5-name.png)

6. When the new instance is **Running**, choose **Networking** in the left menu and open the static IP `fcj-wordpress-ip`. Choose **Detach** and confirm. Then, under **Attach to an instance**, choose `fcj-wordpress-2gb` and choose **Attach**.

![Attach to the new instance](/images/6/6-attach.png)

7. The static IP is now attached to `fcj-wordpress-2gb`. Open `http://<static-ip>/`: the blog loads from the new instance with all its content, because the content is in the managed database.

![Static IP moved](/images/6/7-attached.png)

8. Open the **Networking** tab of `fcj-wordpress-2gb` and check the firewall rules. If the SSH rule is open to any address again, restrict it as in [chapter 4](../4-secure-the-applications/).

9. When the new instance works, delete the old instance `fcj-wordpress` (**Instances** > ⋮ > **Delete**) so that you stop paying for it.
