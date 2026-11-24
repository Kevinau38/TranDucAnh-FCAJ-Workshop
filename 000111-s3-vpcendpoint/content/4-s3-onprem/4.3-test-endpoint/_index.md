---
title : "Test the Interface endpoint"
weight : 3
chapter : false
pre : " <b> 4.3 </b> "
---


#### Get the endpoint DNS name

1. Open the endpoint **s3-interface-endpoint**. On the **Details** tab, copy the first (Regional) DNS name, for example `*.vpce-0123...-abcd.s3.<region>.vpce.amazonaws.com`. Remove the leading `*`.

![dns name](/images/4-s3-onprem/4.3-dns-name.png)

#### Upload from the on-premises instance

1. In **Session Manager**, start a session to **Test-Interface-Endpoint** (it runs in VPC On-prem).
2. Run the following commands. Replace `<Regional-DNS-Name>` with the value you copied (without `*`) and use your bucket name:

```bash
cd ~ && fallocate -l 1G testfile2.xyz && ls -lh testfile2.xyz
aws s3 cp --endpoint-url https://bucket.<Regional-DNS-Name> testfile2.xyz s3://<your-bucket-name> --region <your-region>
getent hosts bucket.<Regional-DNS-Name>
```

![upload](/images/4-s3-onprem/4.3-upload.png)

+ `--endpoint-url` tells the AWS CLI to send the request to the interface endpoint instead of the public S3 endpoint.
+ `getent hosts` shows that the endpoint name resolves to **private IPs in VPC Cloud** (10.10.x.x) – the traffic travels over the hybrid link (peering), not the Internet.

#### Check the object

Open the bucket in the S3 console: **testfile2.xyz** has been added.

![objects](/images/4-s3-onprem/4.3-objects.png)

{{% notice tip %}}
Keep the session open – you will use it again in sections 4.4 and 5.
{{% /notice %}}
