---
title : "Clean up resources"
weight : 8
chapter : false
pre : " <b> 8. </b> "
---

Delete everything you created so that nothing keeps running or storing data. A CloudFront distribution must be disabled and fully deployed before it can be deleted, so the script waits several minutes at that step. A user pool created from the console also has a domain, which must be deleted before the pool.

1. Create and run the cleanup script:

```bash
cat > ~/cleanup.sh <<'EOF'
source ~/chat.env
# 1. CloudFront: disable, wait, delete, then delete the OAC
aws cloudfront get-distribution-config --id $DIST_ID > d.json && E=$(jq -r .ETag d.json)
jq '.DistributionConfig.Enabled=false | .DistributionConfig' d.json > d2.json
aws cloudfront update-distribution --id $DIST_ID --if-match $E --distribution-config file://d2.json >/dev/null && echo "CloudFront disabled, waiting..."
aws cloudfront wait distribution-deployed --id $DIST_ID
E=$(aws cloudfront get-distribution --id $DIST_ID --query ETag --output text)
aws cloudfront delete-distribution --id $DIST_ID --if-match $E && echo "Distribution deleted"
for O in $(aws cloudfront list-origin-access-controls --query "OriginAccessControlList.Items[?contains(Name,'$BUCKET')].Id" --output text); do
  aws cloudfront delete-origin-access-control --id $O --if-match $(aws cloudfront get-origin-access-control --id $O --query ETag --output text) && echo "OAC $O deleted"
done
# 2. API Gateway, Lambda functions and their log groups
aws apigateway delete-rest-api --rest-api-id $API_ID && echo "API deleted"
for f in Chat-Proxy Chat-Conversation-GET Chat-Conversation-POST Chat-Messages-GET Chat-Messages-POST Chat-Users-GET; do
  aws lambda delete-function --function-name $f; aws logs delete-log-group --log-group-name /aws/lambda/$f 2>/dev/null; echo "Lambda $f deleted"
done
# 3. DynamoDB, Cognito (domain first), S3
aws dynamodb delete-table --table-name Chat-Messages >/dev/null; aws dynamodb delete-table --table-name Chat-Conversations >/dev/null; echo "Tables deleted"
aws cognito-idp update-user-pool --user-pool-id $POOL_ID --deletion-protection INACTIVE 2>/dev/null
D=$(aws cognito-idp describe-user-pool --user-pool-id $POOL_ID --query UserPool.Domain --output text)
[ "$D" != "None" ] && aws cognito-idp delete-user-pool-domain --user-pool-id $POOL_ID --domain $D
aws cognito-idp delete-user-pool --user-pool-id $POOL_ID && echo "User pool deleted"
aws s3 rb s3://$BUCKET --force && echo "Bucket deleted"
# 4. IAM roles
for r in FCJ-ChatApp-Lambda-Role FCJ-ChatApp-Cognito-Role; do
  for p in $(aws iam list-role-policies --role-name $r --query PolicyNames --output text); do aws iam delete-role-policy --role-name $r --policy-name $p; done
  aws iam detach-role-policy --role-name $r --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
  aws iam delete-role --role-name $r && echo "Role $r deleted"
done
EOF
bash ~/cleanup.sh
```

2. Check that nothing is left:

```bash
source ~/chat.env
echo "CloudFront: $(aws cloudfront list-distributions --query "DistributionList.Items[?Id=='$DIST_ID'].Id" --output text)"
echo "OAC: $(aws cloudfront list-origin-access-controls --query "OriginAccessControlList.Items[?contains(Name,'$BUCKET')].Id" --output text)"
echo "API: $(aws apigateway get-rest-apis --query "items[?name=='Chat-API'].id" --output text)"
echo "Lambda: $(aws lambda list-functions --query "Functions[?starts_with(FunctionName,'Chat-')].FunctionName" --output text)"
echo "Logs: $(aws logs describe-log-groups --log-group-name-prefix /aws/lambda/Chat- --query 'logGroups[].logGroupName' --output text)"
echo "Tables: $(aws dynamodb list-tables --query "TableNames[?starts_with(@,'Chat-')]" --output text)"
echo "Pool: $(aws cognito-idp list-user-pools --max-results 60 --query "UserPools[?Id=='$POOL_ID'].Id" --output text)"
echo "Bucket: $(aws s3api list-buckets --query "Buckets[?Name=='$BUCKET'].Name" --output text)"
echo "Roles: $(aws iam list-roles --query "Roles[?starts_with(RoleName,'FCJ-ChatApp')].RoleName" --output text)"
```

   - (1) Every line is empty: the distribution, OAC, API, functions, log groups, tables, user pool, bucket and roles are all gone.

   ![Cleanup](/images/8-cleanup/8-cleanup.png)
