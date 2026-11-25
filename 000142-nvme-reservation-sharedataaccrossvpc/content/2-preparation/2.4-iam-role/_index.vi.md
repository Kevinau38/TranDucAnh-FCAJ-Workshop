---
title : "Tạo IAM role"
weight : 4
chapter : false
pre : " <b> 2.4 </b> "
---

Các instance cần managed policy `AmazonSSMManagedInstanceCore` để đăng ký với Systems Manager. Bạn gắn policy này vào một role và một instance profile cùng tên `FCJ-NVMe-EC2-Role`.

1. Trong CloudShell, chạy:

   ```bash
   echo '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":"ec2.amazonaws.com"},"Action":"sts:AssumeRole"}]}' > ec2-trust.json
   aws iam create-role --role-name FCJ-NVMe-EC2-Role --assume-role-policy-document file://ec2-trust.json ${BOUNDARY:+--permissions-boundary $BOUNDARY} --query Role.Arn --output text
   aws iam attach-role-policy --role-name FCJ-NVMe-EC2-Role --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
   aws iam create-instance-profile --instance-profile-name FCJ-NVMe-EC2-Role --query InstanceProfile.Arn --output text
   aws iam add-role-to-instance-profile --instance-profile-name FCJ-NVMe-EC2-Role --role-name FCJ-NVMe-EC2-Role && echo PROFILE-OK
   ```

   - (1) Role đã được tạo.
   - (2) Instance profile đã được tạo.
   - (3) Role đã được thêm vào instance profile.

   ![IAM role và instance profile](/images/2-preparation/2.4-iam-role.png)

{{% notice note %}}
`${BOUNDARY:+...}` chỉ thêm `--permissions-boundary` khi biến `BOUNDARY` có giá trị. Nếu tài khoản của bạn bắt buộc role mới phải có permissions boundary, hãy đặt biến trước, ví dụ `BOUNDARY=arn:aws:iam::<account-id>:policy/<boundary-name>`. Nếu không thì để trống.
{{% /notice %}}
