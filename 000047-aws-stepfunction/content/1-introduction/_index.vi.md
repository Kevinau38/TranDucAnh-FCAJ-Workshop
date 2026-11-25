---
title : "Giới thiệu"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

#### Điều phối service

Điều phối (orchestration) là cách phối hợp các service phân tán từ một workflow trung tâm, giống nhạc trưởng chỉ huy từng nhạc công trong dàn nhạc. Logic của quy trình nằm ở một chỗ thay vì rải rác trong từng service, nên bạn theo dõi được cả quy trình từ đầu đến cuối trên một giao diện. Cách này phù hợp khi các bước cần chạy lại, cần xử lý lỗi, chạy song song, hoặc quy trình phải chờ một việc gì đó rồi mới đi tiếp.

#### Nghiệp vụ

Workshop mô phỏng một phần nghiệp vụ ngân hàng. Khách hàng gửi đơn mở tài khoản, đơn được kiểm tra trước khi duyệt.

- Service **Account Applications**: lưu đơn mới, đánh dấu đơn cần xem xét, duyệt hoặc từ chối đơn, liệt kê đơn theo trạng thái.
- Service **Data Checking**: kiểm tra tên và địa chỉ người đăng ký bằng vài quy tắc đơn giản.
- Service **Accounts**: mở tài khoản khi đơn được duyệt. Workshop không xây dựng service này.

#### Workflow

Với mỗi đơn mới, workflow sẽ:

1. Kiểm tra tên và địa chỉ của người đăng ký.
2. Tự động duyệt đơn nếu cả hai đều hợp lệ. Nếu không kiểm tra được, đánh dấu đơn là không xử lý được và dừng lại.
3. Ngược lại, đánh dấu đơn cần xem xét và tạm dừng.
4. Chờ người kiểm duyệt duyệt hoặc từ chối.
5. Duyệt hoặc từ chối đơn theo quyết định đó.

#### Kiến trúc

![Kiến trúc](/images/1-introduction/architecture.png)

- **AWS CloudShell** chạy AWS CLI và AWS SAM CLI. Không cần IDE hay EC2 instance.
- **AWS SAM** đóng gói code vào một bucket S3 private và tạo CloudFormation stack gồm các Lambda function, bảng DynamoDB, state machine và hai IAM role.
- `SubmitApplication` lưu đơn mới và khởi chạy một execution của state machine.
- State machine gọi `DataChecking`, `FlagApplication`, `ApproveApplication` và `RejectApplication`. Khi đơn cần xem xét, state machine chờ một **task token**; `ReviewApplication` gửi quyết định về bằng `SendTaskSuccess`.

#### Trước khi bắt đầu

- Dùng Region **Asia Pacific (Tokyo) ap-northeast-1**.
- Nếu tài khoản bắt buộc gắn **permissions boundary** cho IAM role mới, các lệnh ở phần 2 sẽ truyền boundary vào template. Nếu không, để trống tham số này.

{{% notice warning %}}
Chi phí: Lambda, Step Functions (Standard), DynamoDB on-demand và S3 đều tính theo request. Chạy hết workshop chỉ tốn vài cent. Làm phần 7 ngay khi xong.
{{% /notice %}}
