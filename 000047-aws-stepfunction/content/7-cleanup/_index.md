---
title : "Clean up resources"
weight : 7
chapter : false
pre : " <b> 7. </b> "
---

Delete everything you created. `sam delete` removes the CloudFormation stack (state machine, functions, table, the two `FCJ-SFN-*` roles) and the packages in the bucket. The bucket itself, the console role and the Lambda log groups are not part of the stack.

1. Delete the resources:

```bash
cd ~/workshop-dir
sam delete --stack-name step-functions-workshop --region ap-northeast-1 --no-prompts
aws s3 rb s3://$(grep s3_bucket samconfig.toml | cut -d'"' -f2) --force
aws iam delete-role --role-name FCJ-SFN-Console-Role
for g in $(aws logs describe-log-groups --log-group-name-prefix /aws/lambda/sfn-workshop- --query 'logGroups[].logGroupName' --output text); do
  aws logs delete-log-group --log-group-name $g && echo "deleted $g"
done
cd ~ && rm -rf ~/workshop-dir
```

2. Check that nothing is left:

```bash
echo "Stack: $(aws cloudformation describe-stacks --stack-name step-functions-workshop --query 'Stacks[0].StackStatus' --output text 2>/dev/null)"
echo "State machines: $(aws stepfunctions list-state-machines --query "stateMachines[?contains(name,'Application')].name" --output text)"
echo "Lambda: $(aws lambda list-functions --query "Functions[?starts_with(FunctionName,'sfn-workshop-')].FunctionName" --output text)"
echo "Logs: $(aws logs describe-log-groups --log-group-name-prefix /aws/lambda/sfn-workshop- --query 'logGroups[].logGroupName' --output text)"
echo "Tables: $(aws dynamodb list-tables --query "TableNames[?starts_with(@,'StepFunctionWorkshop')]" --output text)"
echo "Bucket: $(aws s3api list-buckets --query "Buckets[?starts_with(Name,'fcj-sfn-sam')].Name" --output text)"
echo "Roles: $(aws iam list-roles --query "Roles[?starts_with(RoleName,'FCJ-SFN')].RoleName" --output text)"
```

   - (1) Every line is empty.

   ![Verify](/images/7-cleanup/7-verify.png)
