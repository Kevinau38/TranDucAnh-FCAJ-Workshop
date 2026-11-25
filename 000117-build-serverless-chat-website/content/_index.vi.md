---
title : "Xây dựng ứng dụng chat serverless hoàn chỉnh"
weight : 1
chapter : false
---

# Xây dựng ứng dụng chat serverless hoàn chỉnh

#### Tổng quan

Trong workshop này bạn xây dựng một website chat nhỏ mà không phải quản lý server nào. Bạn bắt đầu từ vài file HTML tĩnh rồi phát triển dần, đến cuối bài người dùng có thể đăng ký, đăng nhập và chat với nhau.

- File của website nằm trong một **bucket Amazon S3 private** và đến tay người dùng qua **Amazon CloudFront**.
- **Amazon API Gateway** và **AWS Lambda** xử lý request từ trình duyệt.
- **Amazon DynamoDB** lưu cuộc trò chuyện và tin nhắn.
- **Amazon Cognito** quản lý người dùng và bảo vệ API.

#### Nội dung

Workshop có tám phần. Bạn làm theo thứ tự, vì phần sau dùng lại tài nguyên của phần trước.

{{% notice info %}}
Đây là phiên bản cập nhật của workshop. Bucket không bao giờ được mở public: website được phục vụ qua CloudFront với Origin Access Control. Lambda dùng runtime Node.js hiện hành, IAM role đặt tên theo quy tắc `FCJ-*` và có permissions boundary, toàn bộ các bước được chạy lại ở **Asia Pacific (Tokyo) ap-northeast-1**.
{{% /notice %}}
