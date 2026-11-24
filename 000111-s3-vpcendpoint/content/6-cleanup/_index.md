---
title : "Clean up resources"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---


Congratulations on completing the lab! You accessed Amazon S3 privately from a VPC with a **Gateway endpoint**, from a simulated on-premises network with an **Interface endpoint** and **Route 53 Resolver**, and restricted access with a **VPC endpoint policy**.

Interface endpoints and Resolver endpoints are billed per hour, so delete everything right away. Follow this order:

1. **Terminate** any open Session Manager sessions.
2. **Route 53 → Resolver → Rules**: select **fcj-s3-forward-rule**, **disassociate** it from **VPC On-prem**, then **Delete** the rule.
3. **Route 53 → Hosted zones → s3.&lt;region&gt;.amazonaws.com**: delete the two **A (alias)** records you created. Leave the NS and SOA records – the zone itself is deleted by CloudFormation.
4. **VPC → Endpoints**: select **s3-gwe** and **s3-interface-endpoint**, choose **Actions → Delete VPC endpoints**, type `delete` and confirm.

![delete endpoints](/images/6-cleanup/6-delete-endpoints.png)
![confirm delete](/images/6-cleanup/6-confirm-delete.png)

5. **S3**: **Empty** and then **Delete** both buckets (`<your-bucket-name>` and `<your-bucket-name>-2`).
6. **CloudFormation**: delete **fcj-s3vpce-dns** first and wait until it is gone, then delete **fcj-s3vpce-base** (it exports values used by the DNS stack). Deleting the base stack also removes the route you added to RT On-prem.

![delete stacks](/images/6-cleanup/6-delete-stacks.png)

{{% notice warning %}}
If a stack fails to delete, open its **Events** tab: usually a resource you created manually (alias records, resolver rule, VPC endpoints) is still attached. Remove it and choose **Retry delete**.
{{% /notice %}}

Equivalent AWS CLI commands (run in CloudShell, same Region):

```bash
aws s3 rb s3://<your-bucket-name> --force
aws s3 rb s3://<your-bucket-name>-2 --force
aws cloudformation delete-stack --stack-name fcj-s3vpce-dns
aws cloudformation wait stack-delete-complete --stack-name fcj-s3vpce-dns
aws cloudformation delete-stack --stack-name fcj-s3vpce-base
```
