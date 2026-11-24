---
title : "Kiểm tra Interface endpoint"
weight : 3
chapter : false
pre : " <b> 4.3 </b> "
---


#### Lấy tên DNS của endpoint

1. Mở endpoint **s3-interface-endpoint**. Tại tab **Details**, sao chép tên DNS đầu tiên (Regional), ví dụ `*.vpce-0123...-abcd.s3.<region>.vpce.amazonaws.com`. Bỏ dấu `*` ở đầu.

![dns name](/images/4-s3-onprem/4.3-dns-name.png)

#### Upload từ instance on-premises

1. Trong **Session Manager**, mở session tới **Test-Interface-Endpoint** (chạy trong VPC On-prem).
2. Chạy các lệnh sau. Thay `<Regional-DNS-Name>` bằng giá trị đã sao chép (không có `*`) và dùng tên bucket của bạn:

```bash
cd ~ && fallocate -l 1G testfile2.xyz && ls -lh testfile2.xyz
aws s3 cp --endpoint-url https://bucket.<Regional-DNS-Name> testfile2.xyz s3://<your-bucket-name> --region <your-region>
getent hosts bucket.<Regional-DNS-Name>
```

![upload](/images/4-s3-onprem/4.3-upload.png)

+ `--endpoint-url` yêu cầu AWS CLI gửi request tới interface endpoint thay vì endpoint public của S3.
+ `getent hosts` cho thấy tên endpoint phân giải ra **IP private trong VPC Cloud** (10.10.x.x) – lưu lượng đi qua kết nối hybrid (peering), không qua Internet.

#### Kiểm tra object

Mở bucket trong S3 console: file **testfile2.xyz** đã được thêm vào.

![objects](/images/4-s3-onprem/4.3-objects.png)

{{% notice tip %}}
Giữ session này mở – bạn sẽ dùng lại ở mục 4.4 và phần 5.
{{% /notice %}}
