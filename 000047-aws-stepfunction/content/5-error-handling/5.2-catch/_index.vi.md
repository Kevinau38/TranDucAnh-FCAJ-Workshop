---
title : "Bắt lỗi của ứng dụng"
weight : 2
chapter : false
pre : " <b> 5.2 </b> "
---

Có những cái tên không kiểm tra được. Function Data Checking mô phỏng việc này: khi tên chứa `UNPROCESSABLE_DATA`, function ném lỗi `UnprocessableDataException`. Thay vì thất bại, workflow nên đánh dấu đơn là không xử lý được.

- **Check Name** có thêm `Catch` cho `UnprocessableDataException`. Thông tin lỗi được lưu vào `$.error-info` và execution chuyển sang state mới **Flag Application As Unprocessable**.
- State đó gọi `FlagApplication` với `flagType` = `UNPROCESSABLE_DATA` cùng thông tin lỗi. Function lưu thông báo lỗi làm lý do.

1. Thay definition rồi deploy:

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
            "Retry": [
                {
                    "ErrorEquals": [
                        "Lambda.ServiceException",
                        "Lambda.AWSLambdaException",
                        "Lambda.SdkClientException",
                        "Lambda.TooManyRequestsException"
                    ]
                }
            ],
            "Catch": [
                {
                    "ErrorEquals": [
                        "UnprocessableDataException"
                    ],
                    "ResultPath": "$.error-info",
                    "Next": "Flag Application As Unprocessable"
                }
            ],
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
            "Retry": [
                {
                    "ErrorEquals": [
                        "Lambda.ServiceException",
                        "Lambda.AWSLambdaException",
                        "Lambda.SdkClientException",
                        "Lambda.TooManyRequestsException"
                    ]
                }
            ],
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
            "Type": "Task",
            "Resource": "arn:aws:states:::lambda:invoke.waitForTaskToken",
            "Parameters": {
                "FunctionName": "${FlagApplicationFunctionName}",
                "Payload": {
                    "id.$": "$.application.id",
                    "flagType": "REVIEW",
                    "taskToken.$": "$$.Task.Token"
                }
            },
            "ResultPath": "$.review",
            "Retry": [
                {
                    "ErrorEquals": [
                        "Lambda.ServiceException",
                        "Lambda.AWSLambdaException",
                        "Lambda.SdkClientException",
                        "Lambda.TooManyRequestsException"
                    ]
                }
            ],
            "Next": "Review Approved?"
        },
        "Review Approved?": {
            "Type": "Choice",
            "Choices": [
                {
                    "Variable": "$.review.decision",
                    "StringEquals": "APPROVE",
                    "Next": "Approve Application"
                },
                {
                    "Variable": "$.review.decision",
                    "StringEquals": "REJECT",
                    "Next": "Reject Application"
                }
            ]
        },
        "Reject Application": {
            "Type": "Task",
            "Parameters": {
                "id.$": "$.application.id"
            },
            "Resource": "${RejectApplicationFunctionArn}",
            "Retry": [
                {
                    "ErrorEquals": [
                        "Lambda.ServiceException",
                        "Lambda.AWSLambdaException",
                        "Lambda.SdkClientException",
                        "Lambda.TooManyRequestsException"
                    ]
                }
            ],
            "End": true
        },
        "Approve Application": {
            "Type": "Task",
            "Parameters": {
                "id.$": "$.application.id"
            },
            "Resource": "${ApproveApplicationFunctionArn}",
            "Retry": [
                {
                    "ErrorEquals": [
                        "Lambda.ServiceException",
                        "Lambda.AWSLambdaException",
                        "Lambda.SdkClientException",
                        "Lambda.TooManyRequestsException"
                    ]
                }
            ],
            "End": true
        },
        "Flag Application As Unprocessable": {
            "Type": "Task",
            "Resource": "arn:aws:states:::lambda:invoke",
            "Parameters": {
                "FunctionName": "${FlagApplicationFunctionName}",
                "Payload": {
                    "id.$": "$.application.id",
                    "flagType": "UNPROCESSABLE_DATA",
                    "errorInfo.$": "$.error-info"
                }
            },
            "ResultPath": "$.review",
            "Retry": [
                {
                    "ErrorEquals": [
                        "Lambda.ServiceException",
                        "Lambda.AWSLambdaException",
                        "Lambda.SdkClientException",
                        "Lambda.TooManyRequestsException"
                    ]
                }
            ],
            "End": true
        }
    }
}
EOF
sam build && sam deploy
```

2. Gửi một đơn có tên không xử lý được và liệt kê các đơn loại này:

```bash
cd ~/workshop-dir
aws lambda invoke --function-name sfn-workshop-SubmitApplication --cli-binary-format raw-in-base64-out --payload '{ "name": "UNPROCESSABLE_DATA", "address": "123 Street" }' out.json && cat out.json; echo
aws lambda invoke --function-name sfn-workshop-FindApplications --cli-binary-format raw-in-base64-out --payload '{ "state": "FLAGGED_WITH_UNPROCESSABLE_DATA" }' out.json >/dev/null && jq '.Items[] | {id, state, reason}' out.json
```

   - (1) Đơn đã được gửi.
   - (2) Đơn ở trạng thái `FLAGGED_WITH_UNPROCESSABLE_DATA`, lý do là thông báo lỗi.

   ![Không xử lý được](/images/5-error-handling/5.2-unprocessable.png)

3. Mở execution mới nhất và chọn **Check Name**.
   - (1) **Check Name** hiện trạng thái đã bắt lỗi.
   - (2) Execution chuyển sang **Flag Application As Unprocessable** và thành công.
   - (3) Output của **Check Name** chứa lỗi `UnprocessableDataException` và thông báo trong `error-info`.

   ![Bắt lỗi](/images/5-error-handling/5.2-caught.png)
