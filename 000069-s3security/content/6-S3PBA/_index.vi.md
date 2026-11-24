---
title : "Cấu hình S3 Block Public Access"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---


#### Cấu hình S3 Block Public Access

Viết policy Deny cho mọi trường hợp rất dễ sai sót. **S3 Block Public Access** cung cấp bốn công tắc đơn giản, ghi đè mọi policy hoặc ACL có thể làm dữ liệu bị công khai. Trong phần này bạn thay bucket policy ở phần 5 bằng thiết lập **new ACLs**.

1. Tại tab **Permissions**, mục **Bucket policy**, chọn **Delete**, nhập `confirm` và chọn **Delete**.

![Xóa bucket policy](/images/6-bpa/6-delete-policy.png)

2. Tại **Block public access (bucket settings)**, chọn **Edit**.
3. Tích **Block public access to buckets and objects granted through *new* access control lists (ACLs)**, giữ nguyên hai thiết lập liên quan đến policy, rồi chọn **Save changes**. Nhập `confirm` khi được hỏi.

![Chặn ACL công khai mới](/images/6-bpa/6-bpa-edit.png)

4. Trong terminal, thử upload công khai lần nữa:

```bash
aws s3api put-object --key text01 --body textfile --acl public-read --bucket $bucket
```

Request bị từ chối với thông báo **public ACLs are prevented by the BlockPublicAcls setting in S3 Block Public Access** – không cần bucket policy.

![Bị chặn bởi S3 Block Public Access](/images/6-bpa/6-test.png)

{{% notice info %}}
Trong môi trường production, hãy bật **Block all public access** ở cả cấp tài khoản và cấp bucket trừ khi có lý do cụ thể. Để chuẩn bị cho phần 8, hãy chỉnh lại thiết lập và **bỏ chọn** tùy chọn *new ACLs* vừa bật, để có thể chủ động tạo ACL công khai và phát hiện nó.
{{% /notice %}}
