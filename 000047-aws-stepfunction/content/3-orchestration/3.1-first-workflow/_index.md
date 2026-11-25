---
title : "Create the first state machine"
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---

A state machine is written in **Amazon States Language** (JSON). The simplest state is **Pass**: it copies its input to its output and does no work. It is useful to sketch a workflow before writing any code.

1. A state machine runs with an IAM role. The console offers to create one, but its name would not follow the `FCJ-*` rule, so create the role in CloudShell. It trusts Step Functions and has no permissions yet:

```bash
BOUNDARY=arn:aws:iam::$(aws sts get-caller-identity --query Account --output text):policy/LabPermissionsBoundary
aws iam create-role --role-name FCJ-SFN-Console-Role ${BOUNDARY:+--permissions-boundary $BOUNDARY} \
  --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":"states.amazonaws.com"},"Action":"sts:AssumeRole"}]}' \
  --query 'Role.[RoleName,PermissionsBoundary.PermissionsBoundaryArn]' --output text
```

   - (1) The role `FCJ-SFN-Console-Role` is created with the permissions boundary.

   ![Console role](/images/3-orchestration/3.1-console-role.png)

2. Open the **Step Functions** console, choose **State machines** (1) and **Create state machine**.

   ![State machines](/images/3-orchestration/3.1-sm-list.png)

3. In the dialog:
   - (1) Keep **Create from blank**.
   - (2) **State machine name**: `Process_New_Account_Applications`.
   - (3) **State machine type**: **Standard**.
   - (4) Choose **Continue**.

   ![Create from blank](/images/3-orchestration/3.1-create-blank.png)

4. Choose **Code** (1) and replace the definition with the JSON below (2). The graph shows three Pass states.

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

5. Choose **Config**. In **Permissions**, open **Execution role**, type `FCJ-SFN` to filter the list and choose `FCJ-SFN-Console-Role` (1). Leave the other settings and choose **Create**.

   ![Execution role](/images/3-orchestration/3.1-role.png)

   - (1) The state machine is created.
   - (2) The query language is **JSONPath**, because the definition does not set `QueryLanguage`. All definitions in this workshop use JSONPath.

   ![Created](/images/3-orchestration/3.1-created.png)

{{% notice tip %}}
The full specification of Amazon States Language is at [states-language.net](https://states-language.net/spec.html).
{{% /notice %}}

#### Run the state machine

6. Choose **Execute**. Every execution can receive a JSON input. Enter the input below (1) and choose **Start execution** (2).

```json
{
    "application": {
        "name": "Spock",
        "address": "123 Enterprise Street"
    }
}
```

   ![Start execution](/images/3-orchestration/3.1-start-input.png)

7. The execution **Succeeded** (1). It only moved through the three Pass states.

   ![Succeeded](/images/3-orchestration/3.1-exec-succeeded.png)

8. Scroll to **Graph view** and choose a state, for example **Check Address** (1). **Input/Output** (2) shows that a Pass state returns its input unchanged.

   ![Step input and output](/images/3-orchestration/3.1-step-io.png)
