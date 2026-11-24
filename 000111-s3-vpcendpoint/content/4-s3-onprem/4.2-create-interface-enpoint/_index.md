---
title : "Create an Interface endpoint"
weight : 2
chapter : false
pre : " <b> 4.2 </b> "
---


1. Open **VPC console → Endpoints → Create endpoint**.
2. **Name tag**: `s3-interface-endpoint`, **Type**: **AWS services**.

![name](/images/4-s3-onprem/4.2-name.png)

3. Filter services by `s3` and choose **com.amazonaws.&lt;region&gt;.s3** with type **Interface**.

![service](/images/4-s3-onprem/4.2-service.png)

4. **VPC**: **VPC Cloud**. Under **Additional settings**, make sure **Enable private DNS name** is **not** selected – in this lab DNS is handled by Route 53 in section 4.4.

![vpc](/images/4-s3-onprem/4.2-vpc.png)

5. **Subnets**: select both Availability Zones and the subnets **Cloud Private Subnet 1** and **Cloud Private Subnet 2**. An ENI is created in each subnet for high availability.

![subnets](/images/4-s3-onprem/4.2-subnets.png)

6. **Security groups**: select **fcj-s3vpce-endpoints-sg**. It allows HTTPS (443) from VPC Cloud and from VPC On-prem.

![security group](/images/4-s3-onprem/4.2-sg.png)

7. Keep **Policy: Full access** and choose **Create endpoint**. The endpoint is **Pending** for 1–2 minutes and then becomes **Available**.

![created](/images/4-s3-onprem/4.2-created.png)
