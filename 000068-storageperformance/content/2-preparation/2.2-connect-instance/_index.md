---
title : "Connect to the instance"
weight : 2
chapter : false
pre : " <b> 2.2 </b> "
---


1. Open the [Amazon EC2 console](https://console.aws.amazon.com/ec2/) → **Instances**, select `FCJ-StoragePerf-Instance` and choose **Connect**.

2. On the **Connect to instance** page, (1) choose **SSM Session Manager**, then (2) choose **Connect**.

   ![Connect with Session Manager](/images/2-preparation/2.2-connect.png)

3. A terminal opens in your browser as `ssm-user`. Start a login shell so that the lab variables are loaded, and go to the home directory:

   ```bash
   bash -l
   cd ~
   ```

4. Check the environment:

   ```bash
   echo $bucket; ls /ebs/tutorial/data-1m | wc -l; du -sh /ebs/tutorial/data-1m; df -h /ebs /efs
   ```

   - (1) `$bucket` holds the name of the lab bucket.
   - (2) `/ebs/tutorial/data-1m` has 2,000 files, about 2 GB, and EFS is mounted at `/efs`.

   ![Verify the environment](/images/2-preparation/2.2-verify.png)

{{% notice note %}}
The screenshots of the terminal are shown with a light background to make them easier to read. Your Session Manager terminal has a dark background.
{{% /notice %}}

{{% notice tip %}}
If the session closes, connect again and run `bash -l` and `cd ~` before you continue. The `aws configure set` values from part 3 are saved in `~/.aws/config`, so they are kept.
{{% /notice %}}
