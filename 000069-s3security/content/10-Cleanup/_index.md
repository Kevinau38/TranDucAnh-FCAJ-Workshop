---
title : "Clean up resources"
weight : 10
chapter : false
pre : " <b> 10. </b> "
---


#### Clean up resources

Delete the resources in this order so that nothing keeps generating charges.

#### IAM Access Analyzer

1. IAM console → **Access analyzer** → **Analyzer settings**, select `FCJ-S3-Analyzer` → **Delete**.

#### AWS Config (only if you set it up in section 8)

2. AWS Config → **Rules**, select `s3-bucket-public-read-prohibited` → **Actions** → **Delete rule**.
3. AWS Config → **Settings** → **Edit**, clear **Enable recording** and save. You can also run the following commands in **CloudShell**:

```bash
aws configservice stop-configuration-recorder --configuration-recorder-name default
aws configservice delete-delivery-channel --delivery-channel-name default
aws configservice delete-configuration-recorder --configuration-recorder-name default
```

4. Empty and delete the Config delivery bucket `fcj-config-lab-<suffix>`.

#### VPC endpoint and bucket

5. VPC console → **Endpoints**, select `S3-Endpoint` → **Actions** → **Delete VPC endpoints**.
6. S3 console → open the lab bucket → **Permissions**, delete the bucket policy if one is still attached.
7. Select the lab bucket → **Empty**, type `permanently delete` and confirm. (The CloudFormation stack cannot delete a bucket that still contains objects.)

#### CloudFormation stack

8. Close the Session Manager tab (**Terminate session**).
9. CloudFormation console → select `FCJ-S3-LAB` → **Delete**. The stack deletes the EC2 instance, the IAM role and instance profile, the bucket, the security group and the VPC.
10. Wait for **DELETE_COMPLETE**, then verify that `SID-vpc` and the `sid-security-*` bucket no longer exist.
