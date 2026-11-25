---
title : "Kiểm tra song song"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---

Kiểm tra tên và kiểm tra địa chỉ không phụ thuộc nhau, nên có thể chạy cùng lúc. State **Parallel** chạy nhiều nhánh, chờ tất cả xong rồi trả về một **mảng** chứa output của từng nhánh theo thứ tự.

- **Check Applicant Data** chạy **Check Name** và **Check Address** thành hai nhánh. `ResultPath` của nó là `$.checks`, nên `checks[0]` là kết quả kiểm tra tên và `checks[1]` là kết quả kiểm tra địa chỉ.
- **Review Required?** giờ đọc `$.checks[0].flagged` và `$.checks[1].flagged`.
- `Catch` cho `UnprocessableDataException` chuyển lên state Parallel: lỗi ở nhánh nào cũng dừng cả state Parallel.

1. Thay definition rồi deploy:

```bash
cd ~/workshop-dir
cat > statemachine/account-application-workflow.asl.json <<'EOF'
{
    "StartAt": "Check Applicant Data",
    "States": {
        "Check Applicant Data": {
            "Type": "Parallel",
            "Branches": [
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
                },
                {
                    "StartAt": "Check Address",
                    "States": {
                        "Check Address": {
                            "Type": "Task",
                            "Parameters": {
                                "command": "CHECK_ADDRESS",
                                "data": {
                                    "address.$": "$.application.address"
                                }
                            },
                            "Resource": "${DataCheckingFunctionArn}",
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
            "ResultPath": "$.checks",
            "Next": "Review Required?"
        },
        "Review Required?": {
            "Type": "Choice",
            "Choices": [
                {
                    "Variable": "$.checks[0].flagged",
                    "BooleanEquals": true,
                    "Next": "Pending Review"
                },
                {
                    "Variable": "$.checks[1].flagged",
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

   - (1) State machine được cập nhật.
   - (2) Deploy hoàn tất.

   ![Deploy](/images/6-parallel/6-deploy.png)

2. Gửi một đơn hợp lệ và liệt kê các đơn đã duyệt:

```bash
cd ~/workshop-dir
aws lambda invoke --function-name sfn-workshop-SubmitApplication --cli-binary-format raw-in-base64-out --payload '{ "name": "Spock", "address": "123 Enterprise Street" }' out.json && cat out.json; echo
aws lambda invoke --function-name sfn-workshop-FindApplications --cli-binary-format raw-in-base64-out --payload '{ "state": "APPROVED" }' out.json >/dev/null && jq '.Items[] | {id, name, address, state}' out.json
```

   - (1) Đơn đã được gửi.
   - (2) Đơn được `ApproveApplicationFunction` tự động duyệt.

   ![Đã duyệt](/images/6-parallel/6-approved.png)

3. Mở execution mới nhất và chọn **Review Required?**.
   - (1) **Check Name** và **Check Address** chạy song song trong **Check Applicant Data**.
   - (2) Input của **Review Required?** có `checks` là mảng gồm hai kết quả.
   - (3) Execution đi sang **Approve Application**.

   ![Song song](/images/6-parallel/6-parallel.png)

{{% notice info %}}
Để tìm hiểu thêm, xem [Map state](https://docs.aws.amazon.com/step-functions/latest/dg/state-map.html) để xử lý một danh sách phần tử, các [service integration](https://docs.aws.amazon.com/step-functions/latest/dg/integrate-services.html) gọi hơn 200 dịch vụ AWS mà không cần Lambda, và [Express workflow](https://docs.aws.amazon.com/step-functions/latest/dg/choosing-workflow-type.html) cho tác vụ ngắn, số lượng lớn.
{{% /notice %}}
