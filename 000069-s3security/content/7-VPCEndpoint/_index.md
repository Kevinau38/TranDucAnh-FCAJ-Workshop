---
title : "Restrict access to an S3 VPC endpoint"
weight : 7
chapter : false
pre : " <b> 7. </b> "
---


#### Restrict access to an S3 VPC endpoint

A **gateway VPC endpoint** lets instances in a VPC reach S3 without going through the Internet Gateway. Combined with the `aws:sourceVpce` condition key in a bucket policy, you can make sure objects are only accessible from your VPC.

#### Create the gateway endpoint

1. Open the [VPC console](https://console.aws.amazon.com/vpc/home) → **Endpoints** → **Create endpoint**.
2. Enter the name `S3-Endpoint` and keep **AWS services** selected.

![Endpoint name](/images/7-vpce/7-name.png)

3. In **Services**, filter on `s3` and select **com.amazonaws.<region>.s3** with type **Gateway**.

![S3 gateway service](/images/7-vpce/7-service.png)

4. In **VPC**, select `SID-vpc`. For now, **do not** select any route table – you will add it later to see the difference. Keep **Full access** for the endpoint policy and choose **Create endpoint**.

![No route table selected](/images/7-vpce/7-no-route-table.png)

5. Copy the **VPC endpoint ID** (`vpce-...`).

![Endpoint created](/images/7-vpce/7-created.png)

#### Add the bucket policy

6. In the S3 console, open the bucket → **Permissions** → **Bucket policy** → **Edit**, paste the policy below (replace the bucket name and the endpoint ID) and choose **Save changes**.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyOutsideVpce",
            "Action": "s3:*",
            "Effect": "Deny",
            "Principal": "*",
            "Resource": "arn:aws:s3:::<your-bucket-name>/*",
            "Condition": {
                "StringNotEquals": {
                    "aws:sourceVpce": "<your-vpc-endpoint-id>"
                }
            }
        }
    ]
}
```

![Deny outside the VPC endpoint](/images/7-vpce/7-policy.png)

7. In the terminal, read the object metadata:

```bash
aws s3api head-object --key app1/file1 --bucket $bucket
```

The request fails with **403 Forbidden**: the subnet route table has no route to the endpoint, so the traffic still leaves through the Internet Gateway and does not carry the endpoint ID.

![Request outside the endpoint denied](/images/7-vpce/7-test-fail.png)

#### Associate the route table

8. Back in **Endpoints**, select `S3-Endpoint` → **Actions** → **Manage route tables**.

![Manage route tables](/images/7-vpce/7-actions.png)

9. Select `SID-routes` and choose **Modify route tables**.

![Select SID-routes](/images/7-vpce/7-select-rt.png)

10. Run the same command again. This time it succeeds, because S3 traffic from the subnet now goes through the gateway endpoint.

![Request through the endpoint allowed](/images/7-vpce/7-test-ok.png)

{{% notice warning %}}
This policy also denies object access from the S3 console and from anywhere outside the VPC – including you. Delete it (or the whole stack) when you are done with the lab.
{{% /notice %}}
