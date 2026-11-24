---
title : "Create a Gateway endpoint"
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---


1. Open the **VPC console**, choose **Endpoints → Create endpoint**.
2. **Name tag**: `s3-gwe`. **Type**: **AWS services**.
3. In **Services**, filter by `s3` and choose **com.amazonaws.&lt;region&gt;.s3** with type **Gateway**.

![select gateway](/images/3-s3-vpc/3.1-select-gateway.png)

4. **VPC**: choose **VPC Cloud**.
5. **Route tables**: select **RT Private Cloud** (the table associated with the 2 private subnets – not the main table).
6. **Policy**: keep **Full access**. You will restrict access with an endpoint policy in section 5.

![route table](/images/3-s3-vpc/3.1-route-table.png)

7. Scroll down and choose **Create endpoint**.

![create](/images/3-s3-vpc/3.1-create.png)

8. The endpoint **s3-gwe** is created with status **Available**.

![done](/images/3-s3-vpc/3.1-done.png)

9. Open **Route tables → RT Private Cloud → Routes**. A new route has been added automatically: destination is the S3 **prefix list** (`pl-...`) and target is the gateway endpoint (`vpce-...`).

![prefix list route](/images/3-s3-vpc/3.1-prefix-route.png)

{{% notice info %}}
The route table has no `0.0.0.0/0` route, so instances in these subnets cannot reach the Internet. Only S3 traffic (matched by the prefix list) leaves the VPC – privately, through the gateway endpoint.
{{% /notice %}}
