---
title : "Deploy the CloudFormation template"
weight : 1
chapter : false
pre : " <b> 2.1 </b> "
---


1. Download the template [fcj-storage-performance.yaml](/files/fcj-storage-performance.yaml) to your computer.

2. Open the [AWS CloudFormation console](https://console.aws.amazon.com/cloudformation/) and choose the Region you will use for the lab. Choose **Create stack** → **With new resources (standard)**.

3. On the **Create stack** page:
   - (1) Choose **Upload a template file**.
   - (2) Choose **Choose file** and select `fcj-storage-performance.yaml`.
   - (3) Choose **Next**.

   ![Upload template](/images/2-preparation/2.1-upload-template.png)

4. On the **Specify stack details** page:
   - (1) **Stack name**: `FCJ-StoragePerf`.
   - (2) **InstanceType**: keep `c7g.large`.
   - (3) **PermissionsBoundaryArn**: leave empty. Fill it in only if your account requires a permissions boundary on new IAM roles (the shared lab account used for the screenshots does).
   - Choose **Next**.

   ![Stack details](/images/2-preparation/2.1-stack-details.png)

5. On the **Configure stack options** page, keep the defaults. At the bottom, (1) select **I acknowledge that AWS CloudFormation might create IAM resources with custom names**, then (2) choose **Next**.

   ![Capabilities](/images/2-preparation/2.1-capabilities.png)

6. On the **Review and create** page, choose **Submit**.

   ![Submit](/images/2-preparation/2.1-submit.png)

7. Wait until the stack status is **CREATE_COMPLETE**, about 3 minutes. The instance creates the EBS sample files and mounts EFS before it reports success to CloudFormation.

   ![CREATE_COMPLETE](/images/2-preparation/2.1-complete.png)

8. Open the (1) **Outputs** tab. (2) Note the bucket name, the file system ID and the instance ID.

   ![Outputs](/images/2-preparation/2.1-outputs.png)

The template creates these resources:

| Resource | Name | Notes |
|---|---|---|
| VPC, public subnet, internet gateway | `FCJ-StoragePerf-*` | `10.40.0.0/16`, one subnet in the first Availability Zone |
| S3 gateway endpoint | | S3 traffic from the instance stays on the AWS network |
| Security groups | `FCJ-StoragePerf-Instance-SG`, `FCJ-StoragePerf-EFS-SG` | The instance has no inbound rule; EFS accepts NFS (2049) only from the instance |
| EC2 instance | `FCJ-StoragePerf-Instance` | Amazon Linux 2023 arm64, root volume 30 GB, data volume 20 GB mounted at `/ebs` with 2,000 files of 1 MB |
| EFS file system | `FCJ-StoragePerf-EFS` | General Purpose, Bursting, mounted at `/efs` |
| S3 bucket | `fcj-storageperf-labbucket-...` | Block Public Access on |
| IAM role | `FCJ-StoragePerf-EC2-Role-<region>` | Session Manager, and read/write access to the lab bucket only |
