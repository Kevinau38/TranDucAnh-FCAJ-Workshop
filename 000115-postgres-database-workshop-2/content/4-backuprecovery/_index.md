---
title : "Backup and Recovery"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

#### Overview

A backup is only useful if you know how to restore it. In this chapter, you review and change the automated backups of `rdspg-lab-db`, take a manual snapshot, restore it, recover the database to a point in time just before a table was dropped, and protect the database with **AWS Backup**.

| Backup type | Created by | Retention | Restore to |
|---|---|---|---|
| Automated backup | RDS, once a day in the backup window, plus transaction logs every 5 minutes | 0 to 35 days | Any second within the retention period (point-in-time recovery) |
| Manual snapshot | You, at any time | Until you delete it | The moment the snapshot was taken |
| AWS Backup recovery point | AWS Backup, on demand or on a schedule from a backup plan | Defined by the backup plan | The moment the backup was taken |

{{% notice info %}}
Every restore creates a **new** DB instance with a new endpoint. The source DB instance keeps running. To switch an application to the restored data, you point the application to the new endpoint, or rename the DB instances.
{{% /notice %}}

{{% notice warning %}}
Each restored DB instance is charged per hour. Delete the restored DB instances as soon as you finish each step.
{{% /notice %}}
