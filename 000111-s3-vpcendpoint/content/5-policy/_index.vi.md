---
title : "VPC endpoint policy"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---


**VPC endpoint policy** là một IAM resource policy gắn vào endpoint. Nó kiểm soát principal, action và resource nào có thể được truy cập **thông qua endpoint đó**. Policy này không thay thế IAM policy hay bucket policy – một request phải được tất cả các policy cho phép. Policy mặc định cho phép toàn quyền.

Trong phần này, bạn giới hạn **s3-interface-endpoint** để client từ on-premises chỉ truy cập được một bucket được phê duyệt. Gateway endpoint policy hoạt động hoàn toàn tương tự.

#### Tạo bucket thứ hai

1. Trong S3 console, tạo bucket mới có tên giống bucket đầu tiên thêm hậu tố `-2`.

![bucket 2](/images/5-policy/5-bucket2-name.png)
![bucket 2 created](/images/5-policy/5-bucket2-done.png)

#### Chỉnh sửa endpoint policy

1. Mở **VPC console → Endpoints → s3-interface-endpoint → Policy**. Policy hiện tại cho phép mọi thứ. Chọn **Edit Policy**.

![default policy](/images/5-policy/5-policy-default.png)

2. Chọn **Custom** và dán policy bên dưới (thay tên bucket), rồi chọn **Save**.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowOnlyBucket2",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::<your-bucket-name>-2",
        "arn:aws:s3:::<your-bucket-name>-2/*"
      ]
    }
  ]
}
```

![edit policy](/images/5-policy/5-policy-edit.png)

#### Kiểm tra policy

Trong session **Test-Interface-Endpoint**, chạy:

```bash
aws s3 cp testfile2.xyz s3://<your-bucket-name>-2 --region <your-region>
aws s3 ls s3://<your-bucket-name>-2 --region <your-region>
aws s3 ls s3://<your-bucket-name> --region <your-region>
```

{{% notice note %}}
Thay đổi endpoint policy có thể mất 1–2 phút để có hiệu lực. Nếu lệnh cuối vẫn thành công, hãy đợi một chút rồi chạy lại.
{{% /notice %}}

+ Truy cập **&lt;your-bucket-name&gt;-2** thành công.
+ Truy cập bucket đầu tiên thất bại với lỗi **AccessDenied ... because no VPC endpoint policy allows the s3:ListBucket action**, dù role của instance có `AmazonS3FullAccess`.

![test policy](/images/5-policy/5-test.png)
