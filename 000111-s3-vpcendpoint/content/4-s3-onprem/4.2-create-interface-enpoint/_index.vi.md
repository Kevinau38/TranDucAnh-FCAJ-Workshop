---
title : "Tạo Interface endpoint"
weight : 2
chapter : false
pre : " <b> 4.2 </b> "
---


1. Mở **VPC console → Endpoints → Create endpoint**.
2. **Name tag**: `s3-interface-endpoint`, **Type**: **AWS services**.

![name](/images/4-s3-onprem/4.2-name.png)

3. Lọc service theo `s3` và chọn **com.amazonaws.&lt;region&gt;.s3** có type **Interface**.

![service](/images/4-s3-onprem/4.2-service.png)

4. **VPC**: **VPC Cloud**. Trong **Additional settings**, đảm bảo **Enable private DNS name** **không** được chọn – trong bài lab này DNS được xử lý bằng Route 53 ở mục 4.4.

![vpc](/images/4-s3-onprem/4.2-vpc.png)

5. **Subnets**: chọn cả hai Availability Zone với subnet **Cloud Private Subnet 1** và **Cloud Private Subnet 2**. Mỗi subnet sẽ có một ENI để đảm bảo tính sẵn sàng cao.

![subnets](/images/4-s3-onprem/4.2-subnets.png)

6. **Security groups**: chọn **fcj-s3vpce-endpoints-sg**. Security group này cho phép HTTPS (443) từ VPC Cloud và VPC On-prem.

![security group](/images/4-s3-onprem/4.2-sg.png)

7. Giữ **Policy: Full access** và chọn **Create endpoint**. Endpoint ở trạng thái **Pending** khoảng 1–2 phút rồi chuyển sang **Available**.

![created](/images/4-s3-onprem/4.2-created.png)
