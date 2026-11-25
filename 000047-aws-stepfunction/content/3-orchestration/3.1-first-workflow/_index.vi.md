---
title : "Tạo state machine đầu tiên"
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---

State machine được viết bằng **Amazon States Language** (JSON). State đơn giản nhất là **Pass**: chép input sang output và không làm gì cả. State này hữu ích để phác thảo workflow trước khi viết code.

1. State machine chạy bằng một IAM role. Console có thể tự tạo role nhưng tên role không theo quy tắc `FCJ-*`, nên bạn tạo role trong CloudShell. Role tin cậy Step Functions và chưa có quyền gì:

```bash
BOUNDARY=arn:aws:iam::$(aws sts get-caller-identity --query Account --output text):policy/LabPermissionsBoundary
aws iam create-role --role-name FCJ-SFN-Console-Role ${BOUNDARY:+--permissions-boundary $BOUNDARY} \
  --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":"states.amazonaws.com"},"Action":"sts:AssumeRole"}]}' \
  --query 'Role.[RoleName,PermissionsBoundary.PermissionsBoundaryArn]' --output text
```

   - (1) Role `FCJ-SFN-Console-Role` đã được tạo kèm permissions boundary.

   ![Role cho console](/images/3-orchestration/3.1-console-role.png)

2. Mở console **Step Functions**, chọn **State machines** (1) và **Create state machine**.

   ![State machines](/images/3-orchestration/3.1-sm-list.png)

3. Trong hộp thoại:
   - (1) Giữ **Create from blank**.
   - (2) **State machine name**: `Process_New_Account_Applications`.
   - (3) **State machine type**: **Standard**.
   - (4) Chọn **Continue**.

   ![Tạo mới](/images/3-orchestration/3.1-create-blank.png)

4. Chọn **Code** (1) và thay definition bằng đoạn JSON dưới đây (2). Sơ đồ hiện ba state Pass.

```json
{
    "StartAt": "Check Name",
    "States": {
        "Check Name": {
            "Type": "Pass",
            "Next": "Check Address"
        },
        "Check Address": {
            "Type": "Pass",
            "Next": "Approve Application"
        },
        "Approve Application": {
            "Type": "Pass",
            "End": true
        }
    }
}
```

   ![Definition](/images/3-orchestration/3.1-code.png)

5. Chọn **Config**. Ở **Permissions**, mở **Execution role**, gõ `FCJ-SFN` để lọc rồi chọn `FCJ-SFN-Console-Role` (1). Giữ nguyên các thiết lập khác và chọn **Create**.

   ![Execution role](/images/3-orchestration/3.1-role.png)

   - (1) State machine đã được tạo.
   - (2) Query language là **JSONPath**, vì definition không khai báo `QueryLanguage`. Mọi definition trong workshop đều dùng JSONPath.

   ![Đã tạo](/images/3-orchestration/3.1-created.png)

{{% notice tip %}}
Đặc tả đầy đủ của Amazon States Language có tại [states-language.net](https://states-language.net/spec.html).
{{% /notice %}}

#### Chạy state machine

6. Chọn **Execute**. Mỗi execution có thể nhận một input JSON. Nhập input dưới đây (1) rồi chọn **Start execution** (2).

```json
{
    "application": {
        "name": "Spock",
        "address": "123 Enterprise Street"
    }
}
```

   ![Start execution](/images/3-orchestration/3.1-start-input.png)

7. Execution **Succeeded** (1). Nó chỉ đi qua ba state Pass.

   ![Thành công](/images/3-orchestration/3.1-exec-succeeded.png)

8. Kéo xuống **Graph view** và chọn một state, ví dụ **Check Address** (1). **Input/Output** (2) cho thấy state Pass trả về đúng input của nó.

   ![Input và output](/images/3-orchestration/3.1-step-io.png)
