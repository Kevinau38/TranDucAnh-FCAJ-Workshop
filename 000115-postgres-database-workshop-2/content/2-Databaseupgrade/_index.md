---
title : "Database Upgrade"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

#### Overview

Amazon RDS for PostgreSQL supports two kinds of engine upgrades:

| Upgrade | Example | Compatibility | How RDS applies it |
|---|---|---|---|
| Minor version | 17.9 → 17.11 | Backward compatible, only bug and security fixes | Manually, or automatically in the maintenance window when **Auto minor version upgrade** is on |
| Major version | 17.11 → 18.6 | May change behavior, catalogs, and extensions | Only when you request it. RDS never upgrades the major version automatically |

In this chapter, you upgrade `rdspg-lab-db` in two steps, from PostgreSQL 17.9 to 17.11, then from 17.11 to 18.6, and you check the result after each step.

{{% notice note %}}
This chapter uses the environment from the **Preparation** step: `rdspg-lab-db` runs PostgreSQL **17.9** and the App server `rdspg-app-server` has `psql` installed.
{{% /notice %}}

{{% notice warning %}}
A major version upgrade cannot be rolled back. In a real environment, test the upgrade on a copy of the database first, for example a DB instance restored from a snapshot, and check that your application works with the new version.
{{% /notice %}}
