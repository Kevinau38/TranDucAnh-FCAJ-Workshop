---
title : "Clean up"
weight : 7
chapter : false
pre : " <b> 7. </b> "
---

Delete what this workshop created. Run the following in CloudShell in **ap-southeast-1**:

```bash
export AWS_DEFAULT_REGION=ap-southeast-1
# 1. Well-Architected workloads created by Service Screener
for w in fcj-screener SS_Report; do
  id=$(aws wellarchitected list-workloads --workload-name-prefix $w \
    --query "WorkloadSummaries[?WorkloadName=='$w'].WorkloadId | [0]" --output text)
  [ "$id" != "None" ] && aws wellarchitected delete-workload --workload-id $id && echo "deleted $w"
done
# 2. Empty marker stacks (ssv2-...)
for s in $(aws cloudformation describe-stacks \
    --query "Stacks[?starts_with(StackName,'ssv2-')].StackName" --output text); do
  aws cloudformation delete-stack --stack-name $s && echo "deleted $s"
done
# 3. Demo bucket
for b in $(aws s3api list-buckets \
    --query "Buckets[?starts_with(Name,'fcj-screener-demo')].Name" --output text); do
  aws s3 rb s3://$b --force
done
# 4. Files in CloudShell
deactivate 2>/dev/null; cd ~
rm -rf /tmp/service-screener-v2 /tmp/bin /tmp/lib /tmp/lib64 /tmp/include /tmp/pyvenv.cfg
```

Check that nothing is left. Every command must return `0`:

```bash
aws wellarchitected list-workloads --query "length(WorkloadSummaries[?WorkloadName=='fcj-screener'||WorkloadName=='SS_Report'])"
aws cloudformation describe-stacks --query "length(Stacks[?starts_with(StackName,'ssv2-')])"
aws s3api list-buckets --query "length(Buckets[?starts_with(Name,'fcj-screener')])"
```

Finally, delete `output.zip` and the extracted report from your computer when you no longer need them.

{{% notice warning %}}
Only delete the `SS_Report` workload if you created it in this workshop. If your team already uses a workload with this name, keep it and delete only `fcj-screener`.
{{% /notice %}}
