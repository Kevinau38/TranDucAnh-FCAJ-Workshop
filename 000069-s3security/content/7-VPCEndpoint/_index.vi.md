---
title : "Giới hạn truy cập qua S3 VPC endpoint"
weight : 7
chapter : false
pre : " <b> 7. </b> "
---


#### Giới hạn truy cập qua S3 VPC endpoint

**Gateway VPC endpoint** cho phép instance trong VPC truy cập S3 mà không đi qua Internet Gateway. Kết hợp với condition key `aws:sourceVpce` trong bucket policy, bạn có thể đảm bảo object chỉ truy cập được từ VPC của mình.

#### Tạo gateway endpoint

1. Mở [VPC console](https://console.aws.amazon.com/vpc/home) → **Endpoints** → **Create endpoint**.
2. Nhập tên `S3-Endpoint` và giữ lựa chọn **AWS services**.

![Tên endpoint](/images/7-vpce/7-name.png)

3. Tại **Services**, lọc `s3` và chọn **com.amazonaws.<region>.s3** có loại **Gateway**.

![Dịch vụ S3 gateway](/images/7-vpce/7-service.png)

4. Tại **VPC**, chọn `SID-vpc`. Tạm thời **không** chọn route table nào – bạn sẽ thêm sau để thấy sự khác biệt. Giữ **Full access** cho endpoint policy và chọn **Create endpoint**.

![Không chọn route table](/images/7-vpce/7-no-route-table.png)

5. Copy **VPC endpoint ID** (`vpce-...`).

![Endpoint đã được tạo](/images/7-vpce/7-created.png)

#### Thêm bucket policy

6. Trong S3 console, mở bucket → **Permissions** → **Bucket policy** → **Edit**, dán policy bên dưới (thay tên bucket và endpoint ID) rồi chọn **Save changes**.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyOutsideVpce",
            "Action": "s3:*",
            "Effect": "Deny",
            "Principal": "*",
            "Resource": "arn:aws:s3:::<your-bucket-name>/*",
            "Condition": {
                "StringNotEquals": {
                    "aws:sourceVpce": "<your-vpc-endpoint-id>"
                }
            }
        }
    ]
}
```

![Từ chối truy cập ngoài VPC endpoint](/images/7-vpce/7-policy.png)

7. Trong terminal, đọc metadata của object:

```bash
aws s3api head-object --key app1/file1 --bucket $bucket
```

Request bị lỗi **403 Forbidden**: route table của subnet chưa có route tới endpoint, nên traffic vẫn đi qua Internet Gateway và không mang endpoint ID.

![Request ngoài endpoint bị từ chối](/images/7-vpce/7-test-fail.png)

#### Gắn route table

8. Quay lại **Endpoints**, chọn `S3-Endpoint` → **Actions** → **Manage route tables**.

![Manage route tables](/images/7-vpce/7-actions.png)

9. Chọn `SID-routes` và chọn **Modify route tables**.

![Chọn SID-routes](/images/7-vpce/7-select-rt.png)

10. Chạy lại lệnh trên. Lần này lệnh thành công vì traffic S3 từ subnet đã đi qua gateway endpoint.

![Request qua endpoint được phép](/images/7-vpce/7-test-ok.png)

{{% notice warning %}}
Policy này cũng chặn truy cập object từ S3 console và từ mọi nơi ngoài VPC – kể cả chính bạn. Hãy xóa policy (hoặc xóa cả stack) khi hoàn thành bài lab.
{{% /notice %}}
