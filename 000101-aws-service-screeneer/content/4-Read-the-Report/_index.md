---
title : "Read the report"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

#### Download the report

1. In CloudShell, choose **Actions** > **Download file**.

![Actions menu](/images/4/1-actions.png)

2. Enter `/tmp/service-screener-v2/output.zip` and choose **Download**. The file is saved to the download folder of your browser.

![Download file](/images/4/2-download.png)

3. Extract `output.zip` on your computer. Open the folder `aws/<account-id>/`. It contains:

| File | Content |
|---|---|
| `index.html` | The Cloudscape report (open this one) |
| `index-legacy.html`, `s3.html`, `ec2.html`, ... | The older report, one page per service |
| `CIS.html`, `NIST.html`, `WAFS.html`, ... | One page per framework |
| `api-full.json`, `all.csv`, `workItem.xlsx` | The findings in machine-readable form, for scripts or for tracking fixes |

4. Open `index.html` in your browser.

#### Dashboard

The **Dashboard** shows the total number of findings by severity (1) and by Well-Architected pillar (2). Choose a pillar or a severity to see the matching findings.

![Dashboard](/images/4/3-dashboard.png)

#### Risk Summary

**Risk Summary** ranks what to fix first. **Risk Posture** (1) counts findings and shows how many have a ready-made AWS CLI fix. **Regions by Risk** (2) shows where the findings are and reminds you that Regions you did not scan are not assessed.

![Risk Summary](/images/4/4-risk.png)

#### Service pages

5. Under **Services**, choose **S3**. **Stats** (1) shows how many resources and rules were checked, how many findings are **Quick Wins** (no downtime, no cost), and how many have a CLI fix.

![S3 page](/images/4/5-service.png)

6. Expand a finding, for example **TlsEnforced**. It explains the risk, lists the affected resources (your demo bucket), and links to the AWS documentation.

![Finding details](/images/4/6-finding.png)

{{% notice warning %}}
The CLI commands in the report are generic suggestions. Read them, test them outside production, and take a backup before you apply them.
{{% /notice %}}

#### Frameworks

7. Under **Frameworks**, choose **Overview**. It shows the compliance rate for each framework. **Not Available** means the control cannot be checked automatically from your configuration.

![Framework overview](/images/4/7-frameworks.png)
