---
title : "Prepare and install"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

#### Permissions

The user or role that runs Service Screener needs:

| Permission | Why |
|---|---|
| `ReadOnlyAccess` (AWS managed policy) | Read the configuration of your resources |
| `AWSCloudShellFullAccess` (AWS managed policy) | Use CloudShell |
| `cloudformation:CreateStack`, `cloudformation:DeleteStack` | Service Screener creates an empty "marker" stack named `ssv2-...` for every run, as an audit record |
| Write access to the Well-Architected Tool, for example `WellArchitectedConsoleFullAccess` | Only for [chapter 5](../5-well-architected-integration/) and the default workload described in chapter 3 |
| `s3:CreateBucket`, `s3:PutBucketTagging`, `s3:DeleteBucket` | Only to create and delete the demo bucket |

#### Open CloudShell

1. Sign in to the AWS console and switch to the **Asia Pacific (Singapore)** Region.
2. Choose the **CloudShell** icon in the top bar, or open [CloudShell](https://console.aws.amazon.com/cloudshell/home?region=ap-southeast-1) directly. Wait until the prompt appears.

![CloudShell](/images/2/1-cloudshell.png)

#### Install Service Screener

3. Service Screener needs Python 3.13. Install it:

```bash
sudo yum install python3.13 -y
python3.13 --version
```

4. Create a virtual environment in `/tmp`, download the tool and install its dependencies. The commands finish without errors.

```bash
cd /tmp
python3.13 -m venv .
source bin/activate
python3.13 -m pip install --upgrade pip
rm -rf service-screener-v2
git clone https://github.com/aws-samples/service-screener-v2.git
cd service-screener-v2
pip install -r requirements.txt
python3.13 scripts/unzip_botocore_lambda_runtime.py
```

![Installation finished](/images/2/2-install.png)

{{% notice note %}}
`pip` may print a `BuildBackendWarning` about an editable installation. You can ignore it.
{{% /notice %}}

5. Build the new report interface (**Cloudscape UI**), then create the `screener` alias. The build takes under a minute and ends with `built in ...`.

```bash
cd cloudscape-ui
npm install
npm run build
cd ..
alias screener='python3 $(pwd)/main.py'
```

![Cloudscape UI built](/images/2/3-build.png)

{{% notice warning %}}
CloudShell deletes `/tmp` when the session ends, and an alias lives only in the current terminal. If you come back later or open a new tab, run steps 3–5 again.
{{% /notice %}}
