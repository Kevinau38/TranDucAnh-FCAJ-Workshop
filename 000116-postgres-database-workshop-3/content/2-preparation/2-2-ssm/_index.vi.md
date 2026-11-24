---
title : "AWS Secrets Manager"
weight : 2
chapter : false
pre : " <b> 2.2. </b> "
---

#### Tổng quan

**AWS Secrets Manager** lưu thông tin đăng nhập để bạn thay mật khẩu viết cứng trong code bằng một lệnh gọi API trả về mật khẩu. Vì bạn đã chọn **Managed in AWS Secrets Manager** khi tạo `rdspg-dev-db`, RDS đã tạo secret giúp bạn, luôn đồng bộ nó với database và tự động xoay vòng mật khẩu.

| | Mật khẩu tự quản lý | Quản lý trong Secrets Manager |
|---|---|---|
| Ai tạo mật khẩu | Bạn | RDS |
| Lưu ở đâu | Bất cứ đâu bạn để (file cấu hình, biến môi trường, wiki...) | Secret tên `rds!db-<id>`, mã hoá bằng AWS KMS |
| Xoay vòng | Thủ công | Tự động, mặc định mỗi 7 ngày |
| Ứng dụng lấy bằng cách nào | Chép vào cấu hình | `GetSecretValue`, được IAM cho phép |

#### Xem secret trong console

1. Mở [AWS Secrets Manager console](https://console.aws.amazon.com/secretsmanager/listsecrets):
    - (1) Secret `rds!db-<id>` do RDS tạo. Bạn không tự đặt tên này.
    - (2) Phần mô tả liên kết secret với ARN của DB instance `rdspg-dev-db`.

    ![Secrets](/images/2/2-2/1.png)

2. Chọn secret:
    - (1) Banner cho biết secret do Amazon RDS quản lý: bạn không tự sửa giá trị được, nên mật khẩu trong secret và trong database không bao giờ lệch nhau.
    - (2) Secret được mã hoá bằng KMS key `aws/secretsmanager`.
    - (3) Mở tab **Rotation**.

    ![Chi tiết secret](/images/2/2-2/2.png)

3. Trong tab **Rotation**:
    - (1) **Rotation status** là **Enabled**.
    - (2) **Rotation schedule** là **7 days**.
    - (3) **Next rotation date** cho biết lần tới RDS đổi mật khẩu.

    ![Rotation](/images/2/2-2/3.png)

{{% notice warning %}}
Vì mật khẩu đổi mỗi tuần, ứng dụng không được đọc một lần rồi giữ mãi. Ở chương 3, code đọc secret mỗi khi mở kết nối mới, có cache ngắn, nên việc xoay vòng không làm ứng dụng lỗi.
{{% /notice %}}

#### Đọc secret từ dev server

4. Trong terminal Session Manager (user `ec2-user`), tìm ARN của secret từ DB instance rồi đọc secret. Lệnh chỉ in ra tên user:

    ```bash
    SECRET_ARN=$(aws rds describe-db-instances --db-instance-identifier rdspg-dev-db \
        --query 'DBInstances[0].MasterUserSecret.SecretArn' --output text)
    aws secretsmanager get-secret-value --secret-id "$SECRET_ARN" --query SecretString --output text \
        | jq '{username, password: "(hidden)"}'
    aws secretsmanager list-secrets 2>&1 | cut -c1-110
    ```

    - (1) Role của server đọc được secret: gồm `username` (`postgres`) và `password`.
    - (2) `list-secrets` lỗi **AccessDeniedException**: role chỉ đọc được các secret do RDS quản lý, ngoài ra không làm được gì khác.

    ![Đọc secret](/images/2/2-2/4.png)

{{% notice tip %}}
`MasterUserSecret.SecretArn` là cách đáng tin cậy để tìm secret: tên `rds!db-<id>` do RDS sinh ra và sẽ đổi nếu bạn tạo lại DB instance. Đừng bao giờ in mật khẩu ra log hay terminal.
{{% /notice %}}

#### (Tuỳ chọn) AWS CLI

{{%expand "Bật secret do RDS quản lý cho DB instance đang có" %}}
```bash
aws rds modify-db-instance \
    --db-instance-identifier <your-db-instance> \
    --manage-master-user-password \
    --apply-immediately
```

RDS tạo secret, đặt mật khẩu ngẫu nhiên mới và không còn chấp nhận mật khẩu cũ. Hãy cập nhật mọi ứng dụng đang dùng mật khẩu cũ.
{{% /expand %}}
