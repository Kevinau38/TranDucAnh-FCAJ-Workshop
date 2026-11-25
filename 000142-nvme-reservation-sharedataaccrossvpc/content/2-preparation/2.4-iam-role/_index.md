---
title : "Create the IAM role"
weight : 4
chapter : false
pre : " <b> 2.4 </b> "
---

The instances need the managed policy `AmazonSSMManagedInstanceCore` to register with Systems Manager. You put it in a role and an instance profile with the same name, `FCJ-NVMe-EC2-Role`.

1. In CloudShell, run:

   ```bash
   echo '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":"ec2.amazonaws.com"},"Action":"sts:AssumeRole"}]}' > ec2-trust.json
   aws iam create-role --role-name FCJ-NVMe-EC2-Role --assume-role-policy-document file://ec2-trust.json ${BOUNDARY:+--permissions-boundary $BOUNDARY} --query Role.Arn --output text
   aws iam attach-role-policy --role-name FCJ-NVMe-EC2-Role --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
   aws iam create-instance-profile --instance-profile-name FCJ-NVMe-EC2-Role --query InstanceProfile.Arn --output text
   aws iam add-role-to-instance-profile --instance-profile-name FCJ-NVMe-EC2-Role --role-name FCJ-NVMe-EC2-Role && echo PROFILE-OK
   ```

   - (1) The role is created.
   - (2) The instance profile is created.
   - (3) The role is added to the instance profile.

   ![IAM role and instance profile](/images/2-preparation/2.4-iam-role.png)

{{% notice note %}}
`${BOUNDARY:+...}` adds `--permissions-boundary` only when the variable `BOUNDARY` is set. If your account requires a permissions boundary on new roles, set it first, for example `BOUNDARY=arn:aws:iam::<account-id>:policy/<boundary-name>`. Otherwise leave it empty.
{{% /notice %}}
