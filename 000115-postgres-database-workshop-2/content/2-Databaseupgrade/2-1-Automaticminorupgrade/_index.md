---
title : "Automatic minor version upgrade"
weight : 1
chapter : false
pre : " <b> 2.1. </b> "
---

#### Overview

When **Auto minor version upgrade** is on, RDS upgrades the DB instance to a newer minor version during the **maintenance window**, a weekly 30-minute period that you choose. RDS upgrades the instance automatically only when both of the following are true:

- **Auto minor version upgrade** is enabled.
- A newer minor version of the same major version is marked as the automatic upgrade target by RDS.

`rdspg-lab-db` was created with this option turned off, so the version stays at 17.9 until you upgrade it yourself in the next step.

#### Check the current setting

1. Open the [Amazon RDS console](https://console.aws.amazon.com/rds/), choose **Databases**, then choose `rdspg-lab-db`:
    - (1) Open the **Maintenance & backups** tab.
    - (2) **Auto minor version upgrade** is **Disabled**.
    - (3) **Maintenance window** shows the weekly period in which RDS applies pending maintenance.
    - (4) To change the setting, choose **Modify**. You do this in the next step together with the minor version upgrade.

    ![Maintenance and backups](/images/2/2-1/1.png)

{{% notice info %}}
**Pending maintenance** lists updates that RDS will apply in the next maintenance window, such as operating system patches or a required minor version upgrade. You can apply them immediately with **Apply now**.
{{% /notice %}}

#### (Optional) AWS CLI

You can run the AWS CLI commands in this workshop from [AWS CloudShell](https://console.aws.amazon.com/cloudshell/). CloudShell already has your console credentials and costs nothing.

{{%expand "Check the auto minor version upgrade setting" %}}
```bash
aws rds describe-db-instances \
    --db-instance-identifier rdspg-lab-db \
    --query 'DBInstances[0].[AutoMinorVersionUpgrade,PreferredMaintenanceWindow]' \
    --output table
```
{{% /expand %}}
