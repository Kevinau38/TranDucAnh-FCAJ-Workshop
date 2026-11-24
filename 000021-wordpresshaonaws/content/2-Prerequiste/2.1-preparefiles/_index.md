---
title : "Prepare the installation files in Amazon S3"
weight : 1
chapter : false
pre : " <b> 2.1 </b> "
---

The web servers run in private subnets without a NAT gateway, so they cannot download WordPress from the internet. In this step you download the files once in **AWS CloudShell** and store them in an S3 bucket. Later the instances read the bucket through an S3 gateway endpoint.

1. Open the [AWS Management Console](https://console.aws.amazon.com/), make sure the Region is **Asia Pacific (Tokyo)**, and choose the **CloudShell** icon at the bottom left.

2. Run the following command. It creates a bucket with a unique name, downloads the latest WordPress package and WP-CLI, and uploads both files to the bucket:

   ```bash
   BUCKET=fcj-wordpress-files-$(date +%s) && aws s3 mb s3://$BUCKET && curl -sSLO https://wordpress.org/latest.tar.gz && curl -sSLO https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar && aws s3 cp latest.tar.gz s3://$BUCKET/ --no-progress && aws s3 cp wp-cli.phar s3://$BUCKET/ --no-progress && aws s3 ls s3://$BUCKET/ && echo "BUCKET=$BUCKET"
   ```

3. Check that the bucket contains `latest.tar.gz` and `wp-cli.phar`, and write down the bucket name from the last line. You use it in the next steps.

   ![Upload files to S3](/images/2-prerequisite/2.1-s3-files.png)

{{% notice note %}}
Replace `fcj-wordpress-files-1795416336` in the rest of this lab with the name of your own bucket.
{{% /notice %}}
