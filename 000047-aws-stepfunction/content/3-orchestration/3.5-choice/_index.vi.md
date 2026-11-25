---
title : "Rẽ nhánh bằng state Choice"
weight : 5
chapter : false
pre : " <b> 3.5 </b> "
---

State **Choice** thêm khả năng rẽ nhánh. State có một danh sách quy tắc; mỗi quy tắc so sánh một giá trị trong input và chỉ ra state tiếp theo. Các quy tắc được xét theo thứ tự, quy tắc đầu tiên khớp sẽ được chọn, và `Default` được dùng khi không quy tắc nào khớp.

State mới **Review Required?** chuyển đơn sang **Pending Review** khi `checks.name.flagged` hoặc `checks.address.flagged` là `true`, ngược lại sang **Approve Application**. Tạm thời **Pending Review** là một state Pass. Xem [Choice state](https://docs.aws.amazon.com/step-functions/latest/dg/state-choice.html) để biết mọi toán tử so sánh.

1. Cập nhật definition rồi deploy:

```bash
cd ~/workshop-dir
cat > statemachine/account-application-workflow.asl.json <<'EOF'
{
    "StartAt": "Check Name",
    "States": {
        "Check Name": {
            "Type": "Task",
            "Parameters": {
                "command": "CHECK_NAME",
                "data": {
                    "name.$": "$.application.name"
                }
            },
            "Resource": "${DataCheckingFunctionArn}",
            "ResultPath": "$.checks.name",
            "Next": "Check Address"
        },
        "Check Address": {
            "Type": "Task",
            "Parameters": {
                "command": "CHECK_ADDRESS",
                "data": {
                    "address.$": "$.application.address"
                }
            },
            "Resource": "${DataCheckingFunctionArn}",
            "ResultPath": "$.checks.address",
            "Next": "Review Required?"
        },
        "Review Required?": {
            "Type": "Choice",
            "Choices": [
                {
                    "Variable": "$.checks.name.flagged",
                    "BooleanEquals": true,
                    "Next": "Pending Review"
                },
                {
                    "Variable": "$.checks.address.flagged",
                    "BooleanEquals": true,
                    "Next": "Pending Review"
                }
            ],
            "Default": "Approve Application"
        },
        "Pending Review": {
            "Type": "Pass",
            "End": true
        },
        "Approve Application": {
            "Type": "Pass",
            "End": true
        }
    }
}
EOF
sam build && sam deploy
```

   - (1) State machine được cập nhật.
   - (2) Deploy hoàn tất.

   ![Deploy](/images/3-orchestration/3.5-deploy.png)

2. Chạy execution với một đơn hợp lệ:

   ```json
   { "application": { "name": "Spock", "address": "123 Enterprise Street" } }
   ```

   - (1) **Review Required?** không thấy cờ nào.
   - (2) Execution đi theo **Default** sang **Approve Application**.

   ![Duyệt](/images/3-orchestration/3.5-approve.png)

3. Chạy execution khác với địa chỉ không khớp mẫu `<số> <chữ>` (1):

   ```json
   { "application": { "name": "Spock", "address": "Somewhere" } }
   ```

   ![Input không hợp lệ](/images/3-orchestration/3.5-invalid-input.png)

4. Chọn **Review Required?**.
   - (1) Execution đã sang **Pending Review**.
   - (2) Vì `checks.address.flagged` là `true`.

   ![Chờ xem xét](/images/3-orchestration/3.5-pending.png)

Workflow đã rẽ nhánh đúng. **Approve Application** và **Pending Review** vẫn chưa làm gì; ở phần sau bạn cho chúng gọi các service và chờ người kiểm duyệt.
