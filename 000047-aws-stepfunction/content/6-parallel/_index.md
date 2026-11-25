---
title : "Run the checks in parallel"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---

The name check and the address check do not depend on each other, so they can run at the same time. A **Parallel** state runs several branches, waits for all of them and returns an **array** with the output of each branch, in order.

- **Check Applicant Data** runs **Check Name** and **Check Address** as two branches. Its `ResultPath` is `$.checks`, so `checks[0]` is the name result and `checks[1]` the address result.
- **Review Required?** now reads `$.checks[0].flagged` and `$.checks[1].flagged`.
- The `Catch` for `UnprocessableDataException` moves to the Parallel state: an error in any branch stops the whole Parallel state.

1. Replace the definition and deploy:

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

   - (1) The state machine is updated.
   - (2) The deployment is complete.

   ![Deploy](/images/6-parallel/6-deploy.png)

2. Submit a valid application and list the approved applications:

```bash
cd ~/workshop-dir
aws lambda invoke --function-name sfn-workshop-SubmitApplication --cli-binary-format raw-in-base64-out --payload '{ "name": "Spock", "address": "123 Enterprise Street" }' out.json && cat out.json; echo
aws lambda invoke --function-name sfn-workshop-FindApplications --cli-binary-format raw-in-base64-out --payload '{ "state": "APPROVED" }' out.json >/dev/null && jq '.Items[] | {id, name, address, state}' out.json
```

   - (1) The application is submitted.
   - (2) It was approved automatically by `ApproveApplicationFunction`.

   ![Approved](/images/6-parallel/6-approved.png)

3. Open the latest execution and choose **Review Required?**.
   - (1) **Check Name** and **Check Address** ran in parallel inside **Check Applicant Data**.
   - (2) The input of **Review Required?** has `checks` as an array of two results.
   - (3) The execution went to **Approve Application**.

   ![Parallel](/images/6-parallel/6-parallel.png)

{{% notice info %}}
To go further, read about the [Map state](https://docs.aws.amazon.com/step-functions/latest/dg/state-map.html) to process a list of items, the [service integrations](https://docs.aws.amazon.com/step-functions/latest/dg/integrate-services.html) that call more than 200 AWS services without Lambda, and [Express workflows](https://docs.aws.amazon.com/step-functions/latest/dg/choosing-workflow-type.html) for short, high-volume workloads.
{{% /notice %}}
