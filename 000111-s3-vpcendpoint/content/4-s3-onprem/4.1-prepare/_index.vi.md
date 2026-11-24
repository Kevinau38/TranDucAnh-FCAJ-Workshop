---
title : "Chuẩn bị môi trường"
weight : 1
chapter : false
pre : " <b> 4.1 </b> "
---


#### Triển khai stack DNS

Template thứ hai tạo các thành phần Route 53 dùng ở mục 4.4:

+ Một **private hosted zone** tên `s3.<region>.amazonaws.com`, gắn với VPC Cloud.
+ Một Route 53 Resolver **inbound endpoint** trong VPC Cloud – trả lời các truy vấn DNS đến từ bên ngoài VPC.
+ Một Route 53 Resolver **outbound endpoint** trong VPC On-prem – chuyển tiếp các truy vấn DNS được chọn tới resolver khác.

1. Trong CloudFormation console chọn **Create stack → With new resources (standard)**, upload **fcj-s3vpce-dns.yaml**, đặt tên stack là **fcj-s3vpce-dns** và tạo stack.
2. Stack mất khoảng 3–5 phút. Bạn có thể làm tiếp các bước sau trong lúc stack đang được tạo.

![dns stack](/images/4-s3-onprem/4.1-dns-stack.png)

{{% notice info %}}
Template DNS import ID của VPC, subnet và security group được stack **fcj-s3vpce-base** export, nên bạn không cần nhập tham số nào.
{{% /notice %}}

#### Định tuyến mạng on-premises tới VPC Cloud

Stack base đã tạo kết nối **VPC peering** (*Hybrid link On-prem to Cloud*) và route từ VPC Cloud về 172.16.0.0/16. Bây giờ bạn hoàn thiện "kết nối hybrid" bằng cách thêm route ở phía on-premises.

1. Mở **VPC console → Route tables**, chọn **RT On-prem**, mở tab **Routes** và chọn **Edit routes**.

![rt onprem](/images/4-s3-onprem/4.1-rt-onprem.png)

2. Chọn **Add route**:
   + **Destination**: `10.10.0.0/16` (VPC Cloud)
   + **Target**: **Peering Connection** → `pcx-...` (Hybrid link On-prem to Cloud)
3. Chọn **Save changes**.

![add route](/images/4-s3-onprem/4.1-add-route.png)

4. Route `10.10.0.0/16 → pcx-...` ở trạng thái **Active**.

![route done](/images/4-s3-onprem/4.1-route-done.png)
