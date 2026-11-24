---
title : "Đảm bảo truy cập Hybrid an toàn đến S3 bằng VPC Endpoint"
weight : 1
chapter : false
---

# Đảm bảo truy cập Hybrid an toàn đến S3 bằng VPC Endpoint

#### Tổng quan

**AWS PrivateLink** cung cấp kết nối riêng tư từ VPC và mạng on-premises đến các dịch vụ AWS mà không cần đi qua Internet công cộng.

Trong bài lab này, bạn sẽ tạo, cấu hình và kiểm tra hai loại VPC endpoint cho **Amazon S3**:

+ **Gateway endpoint** – là một đích (target) trong route table dành cho S3 (và DynamoDB). Instance trong VPC truy cập S3 thông qua route prefix list, không cần Internet Gateway hay NAT. Không tính phí theo giờ.
+ **Interface endpoint** – là các network interface (ENI) có IP private nằm trong subnet của bạn. Vì truy cập được qua IP private nên có thể dùng từ **on-premises** khi được kết nối bằng VPN, Direct Connect hoặc (trong bài lab này) VPC peering. Interface endpoint tính phí theo giờ và theo GB.

![overview](/images/1-introduction/architecture.png)

#### Kiến trúc bài lab

| Thành phần | Mục đích |
|---|---|
| **VPC Cloud** (10.10.0.0/16) | Hai private subnet không có Internet. Chứa EC2 **Test-Gateway-Endpoint**, các S3 endpoint và Route 53 Resolver inbound endpoint. |
| **VPC On-prem** (172.16.0.0/16) | Mô phỏng trung tâm dữ liệu của doanh nghiệp. Chứa **Test-Interface-Endpoint** và Route 53 Resolver outbound endpoint. |
| **VPC peering** | Mô phỏng kết nối hybrid riêng tư (VPN / Direct Connect) giữa hai mạng. |
| **SSM interface endpoint** | Cho phép mở shell vào các instance private bằng Session Manager – không cần SSH key hay bastion. |

{{% notice info %}}
Phiên bản gốc của workshop dùng một EC2 strongSwan VPN ở us-east-1. Phiên bản cập nhật này thay VPN bằng **VPC peering** để bài lab chạy nhanh và rẻ hơn; các khái niệm về S3 endpoint hoàn toàn giữ nguyên. Toàn bộ ảnh chụp được thực hiện tại Region **Asia Pacific (Tokyo) ap-northeast-1** – bạn có thể dùng Region khác, miễn là mọi bước đều làm trong cùng một Region.
{{% /notice %}}
