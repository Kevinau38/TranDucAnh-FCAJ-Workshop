---
title : "Create a custom parameter group"
weight : 1
chapter : false
pre : " <b> 6.1. </b> "
---

#### Create the parameter group

1. In the [Amazon RDS console](https://console.aws.amazon.com/rds/home):
    - (1) Choose **Parameter groups** in the navigation pane.
    - (2) Choose **Create parameter group**.

    ![Parameter groups](/images/6/6-1/1.png)

2. In **Parameter group details**:
    - (1) **Parameter group name**: `rdspg-lab-pg18`
    - (2) **Description**: `Custom parameters for rdspg-lab-db (PostgreSQL 18)`
    - (3) **Engine type**: **PostgreSQL**
    - (4) **Parameter group family**: `postgres18`. The family must match the major version of the DB instance.
    - (5) **Type**: **DB Parameter Group**

    Choose **Create** at the bottom of the page.

    ![Parameter group details](/images/6/6-1/2.png)

3. On the **Custom** tab:
    - (1) `rdspg-lab-pg18` appears with the family `postgres18`. A new group starts with the same values as the default group.

    ![Custom parameter groups](/images/6/6-1/3.png)

{{% notice note %}}
When you upgrade a DB instance to a new major version, you need a parameter group of the new family. This is why the major version upgrade in step 2.2 changed the parameter group of `rdspg-lab-db` to `default.postgres18`.
{{% /notice %}}

#### (Optional) AWS CLI

{{%expand "Create the parameter group with the AWS CLI" %}}
```bash
aws rds create-db-parameter-group \
    --db-parameter-group-name rdspg-lab-pg18 \
    --db-parameter-group-family postgres18 \
    --description "Custom parameters for rdspg-lab-db (PostgreSQL 18)"
```
{{% /expand %}}
