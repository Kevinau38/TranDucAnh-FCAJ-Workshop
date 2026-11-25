---
title : "Create one function per method"
weight : 1
chapter : false
pre : " <b> 5.1 </b> "
---

The folder `v8/Lambda` has three functions. They receive a simple event (for example `{"id": "1"}`) instead of the whole HTTP request.

```bash
ROLE_ARN=$(aws iam get-role --role-name FCJ-ChatApp-Lambda-Role --query Role.Arn --output text)
RT=$(aws lambda get-function-configuration --function-name Chat-Proxy --query Runtime --output text)
echo "export ROLE_ARN=$ROLE_ARN RT=$RT" >> ~/chat.env; cd ~/fn
for f in Chat-Conversation-GET Chat-Messages-GET Chat-Messages-POST; do
  cp ~/ws2-chatapp/v8/Lambda/$f.mjs index.mjs && zip -q $f.zip index.mjs
  aws lambda create-function --function-name $f --runtime $RT --handler index.handler --role $ROLE_ARN --zip-file fileb://$f.zip --query '[FunctionName,Runtime,State]' --output text
done
```

- (1) The loop uses the same role and the same runtime as `Chat-Proxy`.
- (2) The three functions are created.

![Three functions](/images/5-break-monolith/5.1-create-functions.png)
