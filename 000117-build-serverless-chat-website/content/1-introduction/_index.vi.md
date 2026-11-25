---
title : "Giới thiệu"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

Ứng dụng chat gồm phần giao diện tĩnh và phần back end serverless. Trình duyệt tải HTML, CSS, JavaScript từ CloudFront, rồi gọi REST API để đọc và ghi tin nhắn.

#### Kiến trúc

![Kiến trúc](/images/1-introduction/architecture.png)

- **Amazon S3** lưu file của website. Bucket chặn toàn bộ truy cập public.
- **Amazon CloudFront** phân phối file qua HTTPS. CloudFront đọc bucket qua **Origin Access Control (OAC)**, nên bucket chỉ tin đúng distribution này.
- **Amazon API Gateway** cung cấp REST API `Chat-API`. Ở cuối bài, một **Cognito authorizer** kiểm tra ID token của mọi request.
- **AWS Lambda** chạy mỗi method của API bằng một function nhỏ.
- **Amazon DynamoDB** có hai bảng: `Chat-Conversations` (ai ở trong cuộc trò chuyện nào) và `Chat-Messages` (tin nhắn).
- **Amazon Cognito** lưu người dùng. Website đăng ký và đăng nhập bằng thư viện JavaScript của Cognito.

#### Mã nguồn

Website và code Lambda lấy từ repository [HoangGuruu/ws2-chatapp](https://github.com/HoangGuruu/ws2-chatapp). Mỗi bước là một thư mục (`v1` đến `v15`). Bạn tải repository này về CloudShell ở phần 2.

#### Trước khi bắt đầu

- Dùng Region **Asia Pacific (Tokyo) ap-northeast-1** cho mọi tài nguyên theo vùng.
- Phần lớn lệnh chạy trong **AWS CloudShell**. Mở CloudShell bằng biểu tượng trên thanh điều hướng của console.
- Nếu account bắt buộc gắn **permissions boundary** cho IAM role mới, đặt biến này trong CloudShell trước phần 3:

  ```bash
  BOUNDARY=arn:aws:iam::<account-id>:policy/<boundary-name>
  ```

  Nếu không có thì để trống. Các lệnh chỉ thêm `--permissions-boundary` khi biến `BOUNDARY` có giá trị.

{{% notice warning %}}
Chi phí: mọi dịch vụ trong bài tính tiền theo request hoặc theo GB. Thử vài giờ chỉ tốn vài cent. Làm phần 8 ngay khi xong.
{{% /notice %}}
