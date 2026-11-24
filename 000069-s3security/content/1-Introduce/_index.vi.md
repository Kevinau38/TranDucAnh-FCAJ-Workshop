---
title : "Giới thiệu"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---


#### Thực hành bảo mật S3

Mặc định, bucket S3 và các object bên trong là private: chỉ tài khoản sở hữu mới truy cập được. Sau đó bạn có thể cấp quyền bằng cách kết hợp:

+ **IAM policy** gắn vào user và role – định nghĩa một identity được phép làm gì.
+ **Bucket policy** – policy dạng resource-based, áp dụng cho mọi request tới bucket, từ bất kỳ principal nào.
+ **Access control list (ACL)** – cơ chế cũ, cấp quyền đọc/ghi cơ bản trên bucket hoặc object. AWS khuyến nghị tắt ACL trừ khi thực sự cần.
+ **S3 Block Public Access** – lớp bảo vệ ở cấp tài khoản và cấp bucket, ghi đè các policy và ACL có thể làm dữ liệu bị công khai.

Trong bài lab này bạn sẽ:

| Phần | Nội dung |
|---|---|
| 3 | Từ chối mọi request không dùng HTTPS |
| 4 | Từ chối upload không dùng mã hóa SSE-S3 |
| 5 | Ngăn client upload object với ACL công khai |
| 6 | Dùng S3 Block Public Access thay cho bucket policy |
| 7 | Chỉ cho phép truy cập object qua S3 VPC gateway endpoint |
| 8 | Phát hiện bucket công khai bằng AWS Config managed rule |
| 9 | Kiểm tra truy cập từ bên ngoài vào S3 bằng IAM Access Analyzer |

![Kiến trúc bài lab](/images/1-introduction/architecture.png)

#### Chi phí

+ EC2 instance là **t3.micro** (nằm trong free tier ở nhiều tài khoản), và S3 gateway endpoint không tính phí theo giờ.
+ AWS Config tính phí theo số configuration item được ghi lại và số lần đánh giá rule; chỉ ghi lại S3 bucket trong một bài lab ngắn tốn vài cent.
+ Analyzer loại **external access** của IAM Access Analyzer không tính thêm phí.
+ Hãy làm phần [dọn dẹp tài nguyên](../10-cleanup/) ngay khi hoàn thành.
