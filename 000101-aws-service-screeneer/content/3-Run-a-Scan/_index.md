---
title : "Run a scan"
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

#### Create a demo resource

To keep the report small and easy to read, you create one S3 bucket with the tag `Project=fcj-screener` and scan only resources with that tag. On your own workloads you usually scan everything.

1. In CloudShell, create the bucket. The bucket name gets a random suffix because S3 bucket names are global.

```bash
export AWS_DEFAULT_REGION=ap-southeast-1
BUCKET=fcj-screener-demo-$(openssl rand -hex 4)
aws s3api create-bucket --bucket $BUCKET \
  --create-bucket-configuration LocationConstraint=ap-southeast-1
aws s3api put-bucket-tagging --bucket $BUCKET \
  --tagging 'TagSet=[{Key=Project,Value=fcj-screener}]'
echo $BUCKET
```

#### Run the scan

2. Scan Amazon S3 and Amazon EC2 in Singapore, only for resources tagged `Project=fcj-screener`:

```bash
screener --regions ap-southeast-1 --services s3,ec2 \
  --tags Project=fcj-screener --disable-custom-pages 1 --beta 1
```

3. The scan takes about 30 seconds here. At the end, Service Screener prints the path of the report, `/tmp/service-screener-v2/output.zip`, and `Scan Completed`.

![Scan completed](/images/3/1-scan-done.png)

{{% notice info %}}
Checks that apply to the whole Region or account, such as **EBS encryption by default**, are not tied to a tagged resource, so they always appear in the report.
{{% /notice %}}

{{% notice warning %}}
Even without the Well-Architected option of [chapter 5](../5-well-architected-integration/), the current version writes its results to a Well-Architected Tool workload named **`SS_Report`** in the scanned Region, and creates it if it does not exist. It also leaves an empty CloudFormation stack named `ssv2-...` for each run. Delete both in [Clean up](../7-clean-up/).
{{% /notice %}}

#### Useful options

| Option | Example | Effect |
|---|---|---|
| `--regions` | `--regions ap-southeast-1,us-east-1` or `--regions ALL` | Regions to scan (required) |
| `--services` | `--services s3,ec2,rds,iam` | Only these services. Without it, all supported services are checked |
| `--tags` | `--tags env=prod%department=hr,coe` | Only resources with these tags. `%` separates keys, `,` separates values |
| `--beta 1` | | Use the new Cloudscape report (requires the build in chapter 2) |
| `--disable-custom-pages 1` | | Skip the Cost Optimization Hub, Trusted Advisor and cross-service pages. Faster, and these pages are account-wide, not filtered by tag |
| `--suppress_file` | `--suppress_file ./suppressions.json` | Hide findings you have accepted |
| `--resume` | `--resume 1` or `--resume 0` | Continue an interrupted scan, or force a fresh one |
| `--sequential 1` | | Run checks one after another instead of in parallel |

If a scan matching the same Regions and services was interrupted, Service Screener asks `Resume from where it left off? [y/n]`. Answer `y` to continue or `n` to start again.
