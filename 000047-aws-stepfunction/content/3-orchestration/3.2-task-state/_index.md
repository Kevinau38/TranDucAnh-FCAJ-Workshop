---
title : "Call Lambda with Task states"
weight : 2
chapter : false
pre : " <b> 3.2 </b> "
---

A **Task** state does the work named in its `Resource` field. Here it calls the Data Checking function. The `Parameters` field builds the function input from the state input: a key ending in `.$` takes its value from a JSONPath, so `"name.$": "$.application.name"` passes the applicant name.

1. Get the ARN of the Data Checking function from the stack outputs:

```bash
cd ~/workshop-dir
REGION=$(grep region samconfig.toml | awk -F\= '{gsub(/"/, "", $2); gsub(/ /, "", $2); print $2}')
STACK_NAME=$(grep stack_name samconfig.toml | awk -F\= '{gsub(/"/, "", $2); gsub(/ /, "", $2); print $2}')
aws cloudformation describe-stacks --region $REGION --stack-name $STACK_NAME --query 'Stacks[0].Outputs[?OutputKey==`DataCheckingFunctionArn`].OutputValue' --output text
```

   - (1) Copy the ARN.

   ![Function ARN](/images/3-orchestration/3.2-arn.png)

2. Open `Process_New_Account_Applications` and choose **Edit** (1).

   ![Edit](/images/3-orchestration/3.2-edit.png)

3. In **Code**, replace the definition with the JSON below.
   - (1)(2) `Resource` still holds a placeholder.
   - (3) The graph marks both Task states as invalid.

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

   ![Placeholder](/images/3-orchestration/3.2-placeholder.png)

4. Replace both `REPLACE_WITH_DATA_CHECKING_LAMBDA_ARN` with the ARN from step 1 (1)(2). The states become **Lambda: Invoke** (3).

   ![ARN set](/images/3-orchestration/3.2-arn-set.png)

5. Choose **Save** (1). The role is not changed: the console does not add permissions to a role you chose yourself. Then choose **Execute** (2).

   ![Saved](/images/3-orchestration/3.2-saved.png)

6. Use the same input as before (1) and choose **Start execution** (2).

   ![Start](/images/3-orchestration/3.2-start.png)

7. This time the execution **Failed** (1).

   ![Failed](/images/3-orchestration/3.2-failed.png)

8. Expand **Cause** (1). The role `FCJ-SFN-Console-Role` is not allowed to perform `lambda:InvokeFunction` on the Data Checking function.

   ![Cause](/images/3-orchestration/3.2-cause.png)

When a state machine runs, it assumes its IAM role, and every call it makes is checked against that role. The role has no permission to invoke the function yet. You fix this in the next step.
