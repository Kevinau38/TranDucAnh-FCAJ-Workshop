---
title : "Chi phí truyền dữ liệu"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---

#### Tổng quan

Chi phí truyền dữ liệu (data transfer) thường bị bỏ qua khi thiết kế kiến trúc, và trong Cost Explorer nó nằm rải rác ở nhiều dịch vụ (ví dụ **EC2 - Other**, Amazon S3 hay Amazon CloudFront) với các usage type chứa `DataTransfer`, `Bytes` hoặc `AWS-Out`. Lọc theo **Usage type group** > *Data Transfer* để xem chúng cùng nhau. Các nguyên tắc chung:

| Lưu lượng | Chi phí |
|---|---|
| Từ internet vào AWS | Miễn phí |
| Từ AWS ra internet | Tính theo GB, tuỳ Region nguồn và dịch vụ. 100 GB đầu tiên mỗi tháng, cộng chung mọi dịch vụ và Region, được miễn phí |
| Giữa các Availability Zone trong cùng Region | Tính theo GB ở mỗi chiều |
| Trong cùng một Availability Zone, dùng IP private | Miễn phí |
| Giữa các Region | Tính theo GB, tuỳ Region nguồn |
| Qua NAT gateway | Phí theo giờ, cộng phí theo mỗi GB xử lý |
| Tới Amazon S3 hoặc DynamoDB qua gateway VPC endpoint | Miễn phí |
| Qua interface VPC endpoint (AWS PrivateLink) | Phí theo giờ cho mỗi Availability Zone, cộng phí theo mỗi GB xử lý |
| Qua AWS Transit Gateway | Phí theo giờ cho mỗi attachment, cộng phí theo mỗi GB xử lý |
| Qua VPC peering trong cùng Availability Zone | Miễn phí; khác Availability Zone thì tính như giữa các AZ |
| AWS Site-to-Site VPN hoặc AWS Direct Connect | Phí theo giờ cho kết nối, cộng phí dữ liệu ra khỏi AWS |

#### Mẹo

- Truy cập Amazon S3 và DynamoDB từ private subnet qua **gateway VPC endpoint** thay vì NAT gateway.
- Đặt các thành phần trao đổi dữ liệu nhiều (ví dụ application server và cache) trong cùng Availability Zone khi yêu cầu về tính sẵn sàng cho phép.
- Dùng **Amazon CloudFront** để phân phối nội dung tới người dùng: dữ liệu từ origin tới CloudFront miễn phí, và CloudFront có free tier riêng.
- Tránh lưu lượng giữa các Region nếu không có yêu cầu nghiệp vụ, ví dụ phục hồi sau thảm hoạ.
- Ước tính chi phí truyền dữ liệu của một thiết kế mới bằng [AWS Pricing Calculator](https://calculator.aws/).

Giá thay đổi theo thời gian và khác nhau giữa các Region: xem [giá data transfer của Amazon EC2](https://aws.amazon.com/ec2/pricing/on-demand/#Data_Transfer) và [giá Amazon VPC](https://aws.amazon.com/vpc/pricing/).

#### Dọn dẹp

Bạn đã xoá tài nguyên duy nhất của workshop, báo cáo `fcj-monthly-cost-by-region`, ở chương 3. Không còn gì cần dọn dẹp.
