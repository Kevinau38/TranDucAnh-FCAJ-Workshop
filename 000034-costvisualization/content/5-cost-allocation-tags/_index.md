---
title : "Cost allocation tags"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

#### Overview

Grouping by service or Region tells you **what** you pay for. **Cost allocation tags** tell you **who** or **which project** it is for: if your resources have tags such as `Project=fcj` or `Env=prod`, you can group and filter Cost Explorer by these tags. To learn how to tag resources, see workshop [000027](https://000027.awsfcaj.com/).

A tag key appears in Cost Explorer only after it is **activated** as a cost allocation tag:

| Type | Created by | Example |
|---|---|---|
| **User-defined** | You, on your resources | `Project`, `Env`, `Owner` |
| **AWS-generated** | AWS services | `aws:createdBy`, `aws:cloudformation:stack-name` |

1. In the navigation pane of **Billing and Cost Management**, choose **Cost Allocation Tags**:
    - (1) The tag keys found on the resources of the account. Only the management account of an organization can activate them.
    - (2) **Status**: **Inactive** tags are not available in Cost Explorer yet.
    - (3) Select a tag key and choose **Activate**.

    ![Cost allocation tags](/images/5/5-1.png)

2. About 24 hours after activation, the tag appears in Cost Explorer: set **Group by** > **Dimension** to **Tag** and choose the tag key, or filter on a tag value. Costs of resources without the tag are shown as **No tag key**.

{{% notice warning %}}
Tags are not applied to costs retroactively: after you activate a tag, only new costs are tagged. You can request a **Backfill tags** (up to 12 months back) on the same page.
{{% /notice %}}
