---
title : "Mô phỏng DNS on-premises"
weight : 4
chapter : false
pre : " <b> 4.4 </b> "
---


Việc thêm `--endpoint-url` vào mọi lệnh là không thực tế với ứng dụng thật. AWS khuyến nghị dùng **DNS**: DNS server on-premises chuyển tiếp truy vấn `s3.<region>.amazonaws.com` tới Route 53 Resolver **inbound endpoint**, endpoint này trả lời từ một private hosted zone trỏ tới interface endpoint.

Trong bài lab, outbound endpoint ở VPC On-prem đóng vai trò DNS server của doanh nghiệp làm nhiệm vụ chuyển tiếp truy vấn.

#### Tạo alias record cho interface endpoint

1. Mở **Route 53 → Hosted zones**, mở private zone **s3.&lt;region&gt;.amazonaws.com** (do stack DNS tạo). Chọn **Create record**.

![hosted zone](/images/4-s3-onprem/4.4-hosted-zone.png)

2. Record 1 – để trống **Record name** (gốc zone), **Record type** A:
   + Bật **Alias**
   + **Route traffic to**: **Alias to VPC endpoint**, chọn Region của bạn
   + Chọn Regional DNS name của **s3-interface-endpoint** (`*.vpce-...s3.<region>.vpce.amazonaws.com`)

![record 1](/images/4-s3-onprem/4.4-record-1.png)

3. Chọn **Add another record**. Record 2 – **Record name** `*`, type A, **Alias** tới cùng VPC endpoint. Wildcard này bao phủ tên bucket dạng virtual-hosted như `<bucket>.s3.<region>.amazonaws.com`.

![record 2](/images/4-s3-onprem/4.4-record-2.png)

4. Chọn **Create records**. Zone đã có hai alias record.

![records](/images/4-s3-onprem/4.4-records-done.png)

#### Tạo Resolver forwarding rule

1. Mở **Route 53 → Resolver → Inbound endpoints**, mở **fcj-inbound-cloud**. Ghi lại hai **IP address** (mỗi subnet một IP).

![inbound ips](/images/4-s3-onprem/4.4-inbound-ips.png)

2. Vào **Resolver → Rules → Create rule**:
   + **Name**: `fcj-s3-forward-rule`
   + **Rule type**: **Forward**
   + **Domain name**: `s3.<region>.amazonaws.com`
   + **VPCs that use this rule**: **VPC On-prem**

![rule 1](/images/4-s3-onprem/4.4-rule-1.png)

   + **Outbound endpoint**: **fcj-outbound-onprem**
   + **Target IP addresses**: hai IP của inbound endpoint, port **53**

![rule 2](/images/4-s3-onprem/4.4-rule-2.png)

3. Chọn **Submit**. Rule chuyển sang trạng thái **Complete**.

![rule done](/images/4-s3-onprem/4.4-rule-done.png)

#### Kiểm tra

Quay lại session **Test-Interface-Endpoint** và chạy:

```bash
getent hosts <your-bucket-name>.s3.<region>.amazonaws.com
aws s3 ls s3://<your-bucket-name> --region <your-region>
```

Tên miền S3 **chuẩn** giờ phân giải ra IP private của interface endpoint, và AWS CLI hoạt động **không cần** `--endpoint-url`.

![test](/images/4-s3-onprem/4.4-test.png)
