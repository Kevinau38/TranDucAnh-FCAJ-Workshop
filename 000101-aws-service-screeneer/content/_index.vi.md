---
title : "AWS Service Screener"
weight : 1
chapter : false
---

# AWS Service Screener

#### Tổng quan

**Service Screener** là công cụ mã nguồn mở của AWS ([aws-samples/service-screener-v2](https://github.com/aws-samples/service-screener-v2)), tự động kiểm tra tài khoản AWS và so sánh cấu hình tài nguyên của bạn với các best practice của AWS và cộng đồng. Công cụ nhóm các phát hiện (finding) theo các trụ cột của **AWS Well-Architected Framework**: Security, Reliability, Operational Excellence, Performance Efficiency và Cost Optimization.

Công cụ chạy trong **AWS CloudShell** và chỉ gọi các API chỉ đọc (`describe`, `get`, `list`), nên không thay đổi tài nguyên của bạn. Kết quả là một báo cáo HTML mà bạn mở trên máy tính của mình.

![Dashboard của Service Screener](/images/4/3-dashboard.png)

Trong workshop này, bạn sẽ:

1. Cài Service Screener trong CloudShell ở Region **Singapore (ap-southeast-1)**.
2. Tạo một tài nguyên demo nhỏ và quét nó.
3. Tải về và đọc báo cáo.
4. Gửi kết quả sang **AWS Well-Architected Tool**.
5. Dọn dẹp tài nguyên.

#### Kiến trúc

Sơ đồ dưới đây cho thấy Service Screener tác động tới những gì trong tài khoản. Công cụ chạy trong CloudShell, đọc bucket demo có gắn tag bằng các API chỉ đọc, ghi kết quả vào một workload của Well-Architected Tool, để lại một CloudFormation stack rỗng cho mỗi lần chạy, và tạo báo cáo để bạn tải về máy.

![Kiến trúc bài lab](/images/architecture.png)

#### Bạn cần chuẩn bị

- Một tài khoản AWS và user hoặc role có các quyền nêu ở [chương 2](2-prepare-and-install/).
- Khoảng 45 phút.

{{% notice info %}}
Chạy Service Screener nằm trong AWS Free Tier. Nếu đã vượt free tier, mỗi lần chạy tốn chưa tới 0,01 USD. Bucket S3 demo để trống nên không tốn phí.
{{% /notice %}}
