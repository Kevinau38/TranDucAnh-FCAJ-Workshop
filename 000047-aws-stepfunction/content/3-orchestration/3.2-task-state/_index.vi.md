---
title : "Gọi Lambda bằng state Task"
weight : 2
chapter : false
pre : " <b> 3.2 </b> "
---

State **Task** thực hiện công việc ghi trong trường `Resource`. Ở đây nó gọi function Data Checking. Trường `Parameters` tạo input cho function từ input của state: khóa nào kết thúc bằng `.$` sẽ lấy giá trị theo JSONPath, nên `"name.$": "$.application.name"` truyền tên người đăng ký.

1. Lấy ARN của function Data Checking từ output của stack:

```bash
cd ~/workshop-dir
REGION=$(grep region samconfig.toml | awk -F\= '{gsub(/"/, "", $2); gsub(/ /, "", $2); print $2}')
STACK_NAME=$(grep stack_name samconfig.toml | awk -F\= '{gsub(/"/, "", $2); gsub(/ /, "", $2); print $2}')
aws cloudformation describe-stacks --region $REGION --stack-name $STACK_NAME --query 'Stacks[0].Outputs[?OutputKey==`DataCheckingFunctionArn`].OutputValue' --output text
```

   - (1) Sao chép ARN.

   ![ARN của function](/images/3-orchestration/3.2-arn.png)

2. Mở `Process_New_Account_Applications` và chọn **Edit** (1).

   ![Edit](/images/3-orchestration/3.2-edit.png)

3. Ở **Code**, thay definition bằng đoạn JSON dưới đây.
   - (1)(2) `Resource` vẫn là chuỗi giữ chỗ.
   - (3) Sơ đồ báo hai state Task chưa hợp lệ.

```json
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
            "Resource": "REPLACE_WITH_DATA_CHECKING_LAMBDA_ARN",
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
            "Resource": "REPLACE_WITH_DATA_CHECKING_LAMBDA_ARN",
            "Next": "Approve Application"
        },
        "Approve Application": {
            "Type": "Pass",
            "End": true
        }
    }
}
```

   ![Chuỗi giữ chỗ](/images/3-orchestration/3.2-placeholder.png)

4. Thay cả hai `REPLACE_WITH_DATA_CHECKING_LAMBDA_ARN` bằng ARN ở bước 1 (1)(2). Hai state chuyển thành **Lambda: Invoke** (3).

   ![Đã điền ARN](/images/3-orchestration/3.2-arn-set.png)

5. Chọn **Save** (1). Role không thay đổi: console không thêm quyền vào role do bạn tự chọn. Sau đó chọn **Execute** (2).

   ![Đã lưu](/images/3-orchestration/3.2-saved.png)

6. Dùng lại input cũ (1) và chọn **Start execution** (2).

   ![Start](/images/3-orchestration/3.2-start.png)

7. Lần này execution **Failed** (1).

   ![Thất bại](/images/3-orchestration/3.2-failed.png)

8. Mở **Cause** (1). Role `FCJ-SFN-Console-Role` không được phép thực hiện `lambda:InvokeFunction` trên function Data Checking.

   ![Nguyên nhân](/images/3-orchestration/3.2-cause.png)

Khi chạy, state machine đảm nhận IAM role của nó, và mọi lời gọi đều được kiểm tra theo role đó. Role chưa có quyền gọi function. Bạn sẽ sửa ở bước tiếp theo.
