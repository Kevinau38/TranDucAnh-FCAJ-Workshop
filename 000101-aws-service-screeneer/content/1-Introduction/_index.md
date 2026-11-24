---
title : "Introduction"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

#### How Service Screener works

1. You start the tool in CloudShell with the Regions and services to check.
2. For each service, it lists the resources and reads their configuration with read-only API calls.
3. It evaluates each resource against a set of rules (checks). For example, for Amazon S3 it checks whether the bucket enforces HTTPS, has versioning, or has a lifecycle policy.
4. It writes the results to `output.zip`: an HTML report, JSON and CSV files, and reports for several compliance frameworks.

#### What the report contains

| Part | What you see |
|---|---|
| **Dashboard** | Number of findings by severity (High, Medium, Low, Informational) and by Well-Architected pillar |
| **Risk Summary** | The findings ranked by impact, by Region and by service |
| **Services** | One page per service with every finding, the affected resources and links to the documentation |
| **Frameworks** | How your account maps to frameworks such as CIS, NIST, the AWS Foundational Technical Review (FTR) and the Well-Architected Framework (WAFS) |
| **Pages** | Optional pages built from Cost Optimization Hub, Trusted Advisor and cross-service findings |

#### Service Screener and the Well-Architected Tool

The **AWS Well-Architected Tool** is a console service where you answer questions about a workload to find risks. Service Screener complements it: it checks the actual configuration of your resources and can fill in part of a Well-Architected review for you ([chapter 5](../5-well-architected-integration/)).

{{% notice warning %}}
The report shows the names and settings of your resources. AWS states that it **must be opened locally and must not be accessible from the internet**. Do not upload it to a public website or a public S3 bucket.
{{% /notice %}}
