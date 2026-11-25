---
title : "Control state input and output"
weight : 4
chapter : false
pre : " <b> 3.4 </b> "
---

Every state receives JSON as input and passes JSON as output to the next state. By default, the output of a Task state is the **result of the task**, here the response of the Lambda function. That is why **Check Address** received `{"flagged":false}` instead of the application.

**ResultPath** tells the state where to put the result inside its input. With `"ResultPath": "$.checks.name"`, the state keeps the original input and adds the result under `checks.name`. You can read more in [Input and output processing](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-input-output-filtering.html).

1. Add `ResultPath` to both Task states and deploy:

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

   - (1) Only the state machine is updated.
   - (2) The deployment is complete.

   ![Deploy](/images/3-orchestration/3.4-deploy.png)

2. Start a new execution with the same input. All states succeed (1).

   ![Succeeded](/images/3-orchestration/3.4-graph.png)

3. Choose **Check Address** (1). Its input keeps the application and already has the result of **Check Name** under `checks.name` (2).

   ![Input](/images/3-orchestration/3.4-input.png)

4. Scroll to **State output**. **Check Address** added its own result under `checks.address` (1).

   ![Output](/images/3-orchestration/3.4-output.png)

The workflow now runs, but it goes straight from **Check Address** to **Approve Application**. An application must only be approved when neither check is flagged.
