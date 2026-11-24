---
title : "Modify the parameters"
weight : 2
chapter : false
pre : " <b> 6.2. </b> "
---

#### Change two parameters

1. Choose `rdspg-lab-pg18`, then choose **Edit**. Enter `log_connections` in **Filter Parameters**:
    - (1) Only `log_connections` is shown. In PostgreSQL 18, its **Data type** is **list**: you choose which phases of a connection to log.
    - (2) In **Value**, enter `receipt,authentication,authorization`.

    ![log_connections](/images/6/6-2/1.png)

{{% notice info %}}
Before PostgreSQL 18, `log_connections` was a Boolean (`1` or `0`). The value `receipt,authentication,authorization` has the same effect as the old value `1`. The value `all` also logs the time that each connection takes to set up.
{{% /notice %}}

2. Replace the filter with `log_min_duration_statement`:
    - (1) Only `log_min_duration_statement` is shown. The value is in milliseconds.
    - (2) In **Value**, enter `4000`.
    - (3) Choose **Save Changes**. RDS saves both values.

    ![log_min_duration_statement](/images/6/6-2/2.png)

#### Compare with the default parameter group

3. Choose **Parameter groups**, select `rdspg-lab-pg18`, then choose **Actions** > **Compare**:
    - (1) Only the two parameters that you changed differ from `default.postgres18`.

    ![Compare](/images/6/6-2/3.png)

    Choose **Close**.

#### (Optional) AWS CLI

{{%expand "Modify the parameters with the AWS CLI" %}}
```bash
aws rds modify-db-parameter-group \
    --db-parameter-group-name rdspg-lab-pg18 \
    --parameters \
        "ParameterName=log_connections,ParameterValue='receipt,authentication,authorization',ApplyMethod=immediate" \
        "ParameterName=log_min_duration_statement,ParameterValue=4000,ApplyMethod=immediate"

aws rds describe-db-parameters \
    --db-parameter-group-name rdspg-lab-pg18 \
    --source user \
    --query 'Parameters[].[ParameterName,ParameterValue,ApplyType]' \
    --output table
```
{{% /expand %}}
