---
title : "Điều phối workflow với AWS Step Functions"
weight : 1
chapter : false
---

# Điều phối workflow với AWS Step Functions

#### Tổng quan

Trong workshop này bạn xây dựng quy trình mở tài khoản ngân hàng đơn giản và điều phối nó bằng **AWS Step Functions**. Các service là những **AWS Lambda** function nhỏ lưu dữ liệu vào **Amazon DynamoDB**. Bạn bắt đầu từ một state machine chưa làm gì, sau đó thêm dần task, quyền, luồng dữ liệu, rẽ nhánh, bước chờ người duyệt, xử lý lỗi và chạy song song.

- **AWS Step Functions** chạy workflow dưới dạng state machine viết bằng Amazon States Language.
- **AWS Lambda** hiện thực hai service Account Applications và Data Checking (Node.js, AWS SDK for JavaScript v3).
- **Amazon DynamoDB** lưu các đơn đăng ký.
- **AWS SAM** triển khai mọi thứ từ một template, chạy trong **AWS CloudShell**.

#### Nội dung

Workshop có bảy phần. Bạn làm theo thứ tự, vì mỗi phần sửa tiếp ứng dụng của phần trước.

{{% notice info %}}
Đây là phiên bản cập nhật của workshop. Bài dùng AWS CloudShell thay cho AWS Cloud9, Lambda chạy Node.js 24 với AWS SDK for JavaScript v3, mọi IAM role được khai báo với tên `FCJ-*` và permissions boundary, toàn bộ các bước được chạy lại ở **Asia Pacific (Tokyo) ap-northeast-1**.
{{% /notice %}}
