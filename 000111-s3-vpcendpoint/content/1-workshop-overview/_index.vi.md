---
title : "Giới thiệu workshop"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---


#### VPC endpoint

VPC endpoint là một thiết bị ảo cho phép tài nguyên trong VPC (và các mạng kết nối với VPC) truy cập các dịch vụ AWS được hỗ trợ một cách **riêng tư**. Lưu lượng giữa endpoint và dịch vụ luôn nằm trong mạng của AWS.

| | Gateway endpoint | Interface endpoint |
|---|---|---|
| Dịch vụ | Amazon S3, DynamoDB | Hầu hết dịch vụ AWS, bao gồm S3 |
| Cách hoạt động | Route prefix list trong route table | ENI có IP private trong subnet |
| Truy cập từ on-premises | Không | Có (VPN, Direct Connect, peering) |
| DNS | Dùng tên public của S3 | Tên DNS riêng của endpoint, hoặc private DNS / Route 53 |
| Chi phí | Không tính phí theo giờ | Theo giờ cho mỗi AZ + theo GB |

#### Bạn sẽ thực hiện

1. Triển khai hạ tầng mạng bằng AWS CloudFormation.
2. Tạo **S3 Gateway endpoint** và upload file từ một instance private không có Internet.
3. Kết nối mạng on-premises mô phỏng, tạo **S3 Interface endpoint** và upload file qua kết nối riêng tư.
4. Dùng **Route 53 Resolver** (inbound + outbound endpoint và forwarding rule) để client on-premises phân giải tên S3 thông thường ra IP private của endpoint.
5. Gắn **VPC endpoint policy** chỉ cho phép một bucket.
6. Dọn dẹp toàn bộ tài nguyên.

Thời gian thực hiện khoảng **60–90 phút**.
