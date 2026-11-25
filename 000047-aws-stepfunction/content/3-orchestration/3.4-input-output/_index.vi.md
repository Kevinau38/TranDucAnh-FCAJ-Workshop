---
title : "Điều khiển input và output của state"
weight : 4
chapter : false
pre : " <b> 3.4 </b> "
---

Mỗi state nhận JSON làm input và chuyển JSON làm output cho state sau. Mặc định, output của state Task là **kết quả của task**, ở đây là phản hồi của Lambda function. Vì vậy **Check Address** nhận được `{"flagged":false}` chứ không phải đơn đăng ký.

**ResultPath** cho state biết đặt kết quả vào đâu trong input. Với `"ResultPath": "$.checks.name"`, state giữ nguyên input ban đầu và thêm kết quả vào `checks.name`. Xem thêm tại [Input and output processing](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-input-output-filtering.html).

1. Thêm `ResultPath` vào hai state Task rồi deploy:

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
            "Next": "Approve Application"
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

   - (1) Chỉ state machine được cập nhật.
   - (2) Deploy hoàn tất.

   ![Deploy](/images/3-orchestration/3.4-deploy.png)

2. Chạy execution mới với input như cũ. Mọi state đều thành công (1).

   ![Thành công](/images/3-orchestration/3.4-graph.png)

3. Chọn **Check Address** (1). Input của nó vẫn giữ đơn đăng ký và đã có kết quả của **Check Name** trong `checks.name` (2).

   ![Input](/images/3-orchestration/3.4-input.png)

4. Kéo xuống **State output**. **Check Address** thêm kết quả của nó vào `checks.address` (1).

   ![Output](/images/3-orchestration/3.4-output.png)

Workflow đã chạy được, nhưng nó đi thẳng từ **Check Address** sang **Approve Application**. Đơn chỉ được duyệt khi không bước kiểm tra nào bị đánh dấu.
