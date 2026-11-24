---
title : "Parameter groups"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---

#### Overview

On your own PostgreSQL server, you change the configuration in `postgresql.conf`. On Amazon RDS, you cannot access that file. Instead, the configuration comes from a **DB parameter group**: a named set of engine parameters that you attach to one or more DB instances.

| Parameter group | Can you edit it? | Use |
|---|---|---|
| Default (`default.postgres18`) | No | Attached automatically when you do not choose a group |
| Custom (`rdspg-lab-pg18`) | Yes | Your own settings, shared by all DB instances that use the group |

Each parameter has an **apply type**:

- **Dynamic**: RDS applies the new value within a few seconds, without a restart.
- **Static**: the new value is applied only after the DB instance is **rebooted**.

In this chapter, you create a custom parameter group for PostgreSQL 18, turn on two logging parameters, attach the group to `rdspg-lab-db`, and check the new values with `SHOW`.

| Parameter | Default | New value | Effect |
|---|---|---|---|
| `log_connections` | empty (off) | `receipt,authentication,authorization` | Logs every connection attempt, the authentication, and the database and user of the session |
| `log_min_duration_statement` | `-1` (off) | `4000` | Logs every statement that runs for 4 seconds or longer, with its duration |

{{% notice tip %}}
Keep one custom parameter group per engine version and per environment (for example `prod-pg18` and `dev-pg18`). Do not edit a parameter group that production uses before you test the change on another DB instance.
{{% /notice %}}
