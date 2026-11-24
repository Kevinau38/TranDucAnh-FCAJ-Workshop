---
title : "Tạo Gateway endpoint"
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---


1. Mở **VPC console**, chọn **Endpoints → Create endpoint**.
2. **Name tag**: `s3-gwe`. **Type**: **AWS services**.
3. Tại **Services**, lọc `s3` và chọn **com.amazonaws.&lt;region&gt;.s3** có type **Gateway**.

![select gateway](/images/3-s3-vpc/3.1-select-gateway.png)

4. **VPC**: chọn **VPC Cloud**.
5. **Route tables**: chọn **RT Private Cloud** (route table gắn với 2 private subnet – không phải main route table).
6. **Policy**: giữ **Full access**. Bạn sẽ giới hạn quyền bằng endpoint policy ở phần 5.

![route table](/images/3-s3-vpc/3.1-route-table.png)

7. Kéo xuống và chọn **Create endpoint**.

![create](/images/3-s3-vpc/3.1-create.png)

8. Endpoint **s3-gwe** được tạo với trạng thái **Available**.

![done](/images/3-s3-vpc/3.1-done.png)

9. Mở **Route tables → RT Private Cloud → Routes**. Một route mới được thêm tự động: đích là **prefix list** của S3 (`pl-...`), target là gateway endpoint (`vpce-...`).

![prefix list route](/images/3-s3-vpc/3.1-prefix-route.png)

{{% notice info %}}
Route table này không có route `0.0.0.0/0`, nên instance trong các subnet này không thể ra Internet. Chỉ lưu lượng S3 (khớp với prefix list) được đi ra khỏi VPC – một cách riêng tư, qua gateway endpoint.
{{% /notice %}}
