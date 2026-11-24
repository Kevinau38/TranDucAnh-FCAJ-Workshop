---
title : "Kiểm tra Gateway endpoint"
weight : 2
chapter : false
pre : " <b> 3.2 </b> "
---


#### Tạo S3 bucket

1. Mở **S3 console → Create bucket**.
2. Giữ **General purpose**, nhập **Bucket name** duy nhất toàn cầu (ví dụ `fcj-s3vpce-<ten-ban>`), giữ nguyên các thiết lập khác và chọn **Create bucket**.

![bucket name](/images/3-s3-vpc/3.2-bucket-name.png)
![bucket created](/images/3-s3-vpc/3.2-bucket-done.png)

#### Kết nối vào instance bằng Session Manager

1. Mở **Systems Manager → Session Manager → Start session**.
2. Chọn **Test-Gateway-Endpoint** (chạy trong VPC Cloud) và chọn **Start session**.

![ssm target](/images/3-s3-vpc/3.2-ssm-target.png)

{{% notice note %}}
Session Manager kết nối tới instance này thông qua các SSM interface endpoint do stack base tạo ra – không cần SSH key, IP public hay bastion host.
{{% /notice %}}

#### Upload file qua gateway endpoint

Chạy các lệnh sau trong session (thay tên bucket và Region của bạn):

```bash
cd ~ && fallocate -l 1G testfile.xyz && ls -lh testfile.xyz
aws s3 cp testfile.xyz s3://<your-bucket-name> --region <your-region>
curl -s -m 5 -o /dev/null https://aws.amazon.com || echo "No Internet access from this subnet"
```

File 1 GB được upload thành công, trong khi request tới một website public bị timeout – chứng minh việc upload đi qua **gateway endpoint** chứ không qua Internet.

![upload](/images/3-s3-vpc/3.2-upload.png)

#### Kiểm tra object

Mở bucket trong S3 console. File **testfile.xyz** (1 GB) đã có trong bucket.

![object](/images/3-s3-vpc/3.2-object.png)

Đóng cửa sổ session khi hoàn tất.
