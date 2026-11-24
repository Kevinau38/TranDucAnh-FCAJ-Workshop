---
title : "Dọn dẹp tài nguyên"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---


Chúc mừng bạn đã hoàn thành bài lab! Bạn đã truy cập Amazon S3 một cách riêng tư từ VPC bằng **Gateway endpoint**, từ mạng on-premises mô phỏng bằng **Interface endpoint** và **Route 53 Resolver**, đồng thời giới hạn quyền truy cập bằng **VPC endpoint policy**.

Interface endpoint và Resolver endpoint tính phí theo giờ, vì vậy hãy xóa toàn bộ tài nguyên ngay. Thực hiện theo thứ tự:

1. **Terminate** các session Session Manager đang mở.
2. **Route 53 → Resolver → Rules**: chọn **fcj-s3-forward-rule**, **disassociate** khỏi **VPC On-prem**, rồi **Delete** rule.
3. **Route 53 → Hosted zones → s3.&lt;region&gt;.amazonaws.com**: xóa hai record **A (alias)** bạn đã tạo. Giữ lại record NS và SOA – zone sẽ được CloudFormation xóa.
4. **VPC → Endpoints**: chọn **s3-gwe** và **s3-interface-endpoint**, chọn **Actions → Delete VPC endpoints**, gõ `delete` và xác nhận.

![delete endpoints](/images/6-cleanup/6-delete-endpoints.png)
![confirm delete](/images/6-cleanup/6-confirm-delete.png)

5. **S3**: **Empty** rồi **Delete** cả hai bucket (`<your-bucket-name>` và `<your-bucket-name>-2`).
6. **CloudFormation**: xóa **fcj-s3vpce-dns** trước và đợi xóa xong, sau đó xóa **fcj-s3vpce-base** (stack này export giá trị cho stack DNS). Xóa stack base cũng xóa luôn route bạn đã thêm vào RT On-prem.

![delete stacks](/images/6-cleanup/6-delete-stacks.png)

{{% notice warning %}}
Nếu stack xóa thất bại, mở tab **Events**: thường là do một tài nguyên bạn tạo thủ công (alias record, resolver rule, VPC endpoint) vẫn còn gắn vào. Xóa tài nguyên đó và chọn **Retry delete**.
{{% /notice %}}

Các lệnh AWS CLI tương đương (chạy trong CloudShell, cùng Region):

```bash
aws s3 rb s3://<your-bucket-name> --force
aws s3 rb s3://<your-bucket-name>-2 --force
aws cloudformation delete-stack --stack-name fcj-s3vpce-dns
aws cloudformation wait stack-delete-complete --stack-name fcj-s3vpce-dns
aws cloudformation delete-stack --stack-name fcj-s3vpce-base
```
