---
title : "Branch with a Choice state"
weight : 5
chapter : false
pre : " <b> 3.5 </b> "
---

A **Choice** state adds branching. It has a list of rules; each rule compares a value of the input and names the next state. Rules are evaluated in order, the first match wins, and `Default` is used when no rule matches.

The new state **Review Required?** sends the application to **Pending Review** when `checks.name.flagged` or `checks.address.flagged` is `true`, otherwise to **Approve Application**. **Pending Review** is a Pass state for now. See [Choice state](https://docs.aws.amazon.com/step-functions/latest/dg/state-choice.html) for all comparison operators.

1. Update the definition and deploy:

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

   - (1) The state machine is updated.
   - (2) The deployment is complete.

   ![Deploy](/images/3-orchestration/3.5-deploy.png)

2. Start an execution with a valid application:

   ```json
   { "application": { "name": "Spock", "address": "123 Enterprise Street" } }
   ```

   - (1) **Review Required?** finds no flag.
   - (2) The execution follows **Default** to **Approve Application**.

   ![Approve](/images/3-orchestration/3.5-approve.png)

3. Start another execution with an address that does not match the pattern `<number> <word>` (1):

   ```json
   { "application": { "name": "Spock", "address": "Somewhere" } }
   ```

   ![Invalid input](/images/3-orchestration/3.5-invalid-input.png)

4. Choose **Review Required?**.
   - (1) The execution went to **Pending Review**.
   - (2) Because `checks.address.flagged` is `true`.

   ![Pending review](/images/3-orchestration/3.5-pending.png)

The workflow now routes applications correctly. **Approve Application** and **Pending Review** still do nothing; in the next part you make them call the services and wait for a reviewer.
