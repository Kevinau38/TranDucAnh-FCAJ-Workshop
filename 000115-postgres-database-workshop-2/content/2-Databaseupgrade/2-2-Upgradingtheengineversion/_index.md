---
title : "Upgrade the engine version"
weight : 2
chapter : false
pre : " <b> 2.2. </b> "
---

#### Overview

You upgrade `rdspg-lab-db` in two steps:

| Step | From | To | What RDS does |
|---|---|---|---|
| Minor version upgrade | 17.9 | 17.11 | Takes a snapshot, upgrades the engine, and restarts the database |
| Major version upgrade | 17.11 | 18.6 | Takes a pre-upgrade snapshot, runs `pg_upgrade`, switches to the `default.postgres18` parameter group, and restarts the database |

During each upgrade the database is unavailable for a few minutes.

{{% notice warning %}}
If the DB instance has read replicas, upgrade them before the source instance for a minor version upgrade. For a major version upgrade, RDS upgrades the in-Region read replicas together with the source instance.
{{% /notice %}}

#### Upgrade the minor version (17.9 → 17.11)

1. Choose `rdspg-lab-db`, then choose **Modify** (as in step 2.1). On the **Modify DB instance: rdspg-lab-db** page, in **Settings**:
    - (1) Open the **DB engine version** list. It shows only the versions that 17.9 can upgrade to.
    - (2) Choose **17.11**.

    ![DB engine version](/images/2/2-2/1.png)

2. Scroll down to **Maintenance**:
    - (1) Select **Enable auto minor version upgrade**, so that RDS installs future minor versions of PostgreSQL 18 for you.
    - (2) Keep the **DB instance maintenance window** unchanged.

    ![Maintenance](/images/2/2-2/2.png)

3. Choose **Continue**. On the summary page:
    - (1) Check the changes: **Engine version** 17.9 → 17.11, **Auto minor version upgrade** No → Yes.
    - (2) Choose **Apply immediately**.
    - (3) Choose **Modify DB instance**.

    ![Summary of modifications](/images/2/2-2/3.png)

{{% notice info %}}
With **Apply during the next scheduled maintenance window**, RDS waits until the maintenance window to apply the change. In this workshop you apply it immediately so you can see the result.
{{% /notice %}}

4. The **Status** of `rdspg-lab-db` changes to **Backing-up**, then **Upgrading**.

    ![Upgrading](/images/2/2-2/4.png)

5. After about 10 minutes, the **Status** returns to **Available**. Open the **Logs & events** tab:
    - (1) In **Recent events**, the event *Database instance engine minor version upgrade complete. Previous version: 17.9.R2. New version: 17.11.R1.* confirms the upgrade.

    ![Minor version upgrade event](/images/2/2-2/5.png)

#### Upgrade the major version (17.11 → 18.6)

6. Choose **Modify** again. In **Settings**:
    - (1) **DB engine version**: **18.6**. From 17.11, the only major version target is 18.6.

    ![DB engine version 18.6](/images/2/2-2/6.png)

7. Scroll down to **Additional configuration** > **Database options**:
    - (1) **DB parameter group** has changed to `default.postgres18`. Each major version needs a parameter group of its own family.

    ![DB parameter group](/images/2/2-2/7.png)

{{% notice note %}}
If you use a custom parameter group, create a copy for the new family (for example `postgres18`) and choose it here. You work with custom parameter groups in chapter 6.
{{% /notice %}}

8. Choose **Continue**:
    - (1) Read the warning: a major version upgrade cannot be reverted, and the database is unavailable for a few minutes.

    ![Upgrade warning](/images/2/2-2/8.png)

9. On the same page:
    - (1) Check the changes: **Engine version** 17.11 → 18.6, **DB parameter group** `default.postgres17` → `default.postgres18`.
    - (2) Choose **Apply immediately**.
    - (3) Choose **Modify DB instance**.

    ![Summary of the major version upgrade](/images/2/2-2/9.png)

10. The **Status** changes to **Upgrading**. A major version upgrade takes about 15 to 20 minutes.

    ![Major version upgrading](/images/2/2-2/10.png)

{{% notice info %}}
Before a major version upgrade, RDS runs a pre-check and automatically takes a snapshot named `rds:preupgrade-rdspg-lab-db-17-11-to-18-6-...`. If the upgrade fails, you can restore the database from this snapshot.
{{% /notice %}}

#### (Optional) AWS CLI

{{%expand "Upgrade the engine version with the AWS CLI" %}}
```bash
# List the versions that the DB instance can upgrade to
aws rds describe-db-engine-versions \
    --engine postgres --engine-version 17.11 \
    --query 'DBEngineVersions[0].ValidUpgradeTarget[].[EngineVersion,IsMajorVersionUpgrade]' \
    --output table

# Minor version upgrade
aws rds modify-db-instance \
    --db-instance-identifier rdspg-lab-db \
    --engine-version 17.11 \
    --auto-minor-version-upgrade \
    --apply-immediately

# Major version upgrade
aws rds modify-db-instance \
    --db-instance-identifier rdspg-lab-db \
    --engine-version 18.6 \
    --db-parameter-group-name default.postgres18 \
    --allow-major-version-upgrade \
    --apply-immediately
```
{{% /expand %}}
