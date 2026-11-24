---
title : "Cost visualization with AWS Cost Explorer"
weight : 1
chapter : false
---

# Cost visualization with AWS Cost Explorer

#### Overview

You cannot optimize what you cannot see. **AWS Cost Explorer** shows the cost and usage of your AWS account as charts and tables, so that you can find which services, Regions, or usage types drive your bill, and follow how they change over time. This follows the **Cost Optimization** pillar of the AWS Well-Architected Framework: *practice cloud financial management* and *expenditure and usage awareness*.

In this workshop you:

| Chapter | You learn to |
|---|---|
| 1. Cost Explorer basics | Read the cost and usage report: time range, granularity, chart types, and the breakdown table |
| 2. Filter and group | Narrow the report to some services and break it down by usage type or Region |
| 3. Saved reports | Save a report to the report library, then delete it |
| 4. Coverage reports | Read the Savings Plans and Reserved Instances coverage reports |
| 5. Cost allocation tags | Use tags to split costs by project or environment |
| 6. Data transfer costs | Understand the data transfer charges of common architectures |

#### What you need

- An AWS account and a user or role that can open **Billing and Cost Management** (for example the AWS managed policy `AWSBillingReadOnlyAccess`). In AWS Organizations, the management account can view the costs of all member accounts.
- The workshop creates nothing except a saved report that you delete at the end, so it has **no cost**: using Cost Explorer in the console is free.

{{% notice note %}}
The screenshots come from a shared lab account, so the amounts and account IDs are blurred. Your own numbers will differ.
{{% /notice %}}
