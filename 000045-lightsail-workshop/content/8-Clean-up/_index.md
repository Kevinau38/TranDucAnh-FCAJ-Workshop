---
title : "Clean up"
weight : 8
chapter : false
pre : " <b> 8. </b> "
---

Delete every resource of this lab so that you stop paying for it. You can use the console or CloudShell.

#### Use the console

1. **Instances**: for each instance (`fcj-wordpress-2gb`, `fcj-prestashop`, and `fcj-wordpress` if it still exists), open the actions menu (⋮) and choose **Delete**.

![Delete an instance](/images/8/1-delete-menu.png)

2. Confirm with **Yes, delete**. Alarms of an instance are deleted with it.

![Confirm deletion](/images/8/2-confirm.png)

3. **Databases**: open `fcj-lightsail-db`, choose the **Delete** tab, and delete the database. You do not need a final snapshot.

4. **Snapshots**: delete `fcj-wordpress-snap-1`.

5. **Networking**: delete the static IPs `fcj-wordpress-ip` and `fcj-prestashop-ip`. A static IP that is not attached is billed.

6. **Alarm notifications**: if you added an email contact, delete it.

#### Use CloudShell

```bash
aws lightsail delete-instance --instance-name fcj-wordpress-2gb
aws lightsail delete-instance --instance-name fcj-prestashop
aws lightsail delete-instance --instance-name fcj-wordpress   # if you did not delete it in chapter 6
aws lightsail delete-relational-database --relational-database-name fcj-lightsail-db --skip-final-snapshot
aws lightsail delete-instance-snapshot --instance-snapshot-name fcj-wordpress-snap-1
aws lightsail release-static-ip --static-ip-name fcj-wordpress-ip
aws lightsail release-static-ip --static-ip-name fcj-prestashop-ip
rm -f lsssh.sh /tmp/ad.json /tmp/lskey /tmp/lskey-cert.pub
```

Check that nothing is left. Every command must return `[]`:

```bash
aws lightsail get-instances --query 'instances[].name'
aws lightsail get-relational-databases --query 'relationalDatabases[].name'
aws lightsail get-static-ips --query 'staticIps[].name'
aws lightsail get-instance-snapshots --query 'instanceSnapshots[].name'
aws lightsail get-alarms --query 'alarms[].name'
```
