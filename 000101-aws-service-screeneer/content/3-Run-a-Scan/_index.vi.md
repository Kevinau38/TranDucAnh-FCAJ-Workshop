---
title : "Chạy quét"
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

#### Tạo tài nguyên demo

Để báo cáo gọn và dễ đọc, bạn tạo một bucket S3 gắn tag `Project=fcj-screener` và chỉ quét các tài nguyên có tag này. Với workload thật, thường bạn sẽ quét toàn bộ.

1. Trong CloudShell, tạo bucket. Tên bucket có hậu tố ngẫu nhiên vì tên bucket S3 là duy nhất trên toàn cầu.

```bash
export AWS_DEFAULT_REGION=ap-southeast-1
BUCKET=fcj-screener-demo-$(openssl rand -hex 4)
aws s3api create-bucket --bucket $BUCKET \
  --create-bucket-configuration LocationConstraint=ap-southeast-1
aws s3api put-bucket-tagging --bucket $BUCKET \
  --tagging 'TagSet=[{Key=Project,Value=fcj-screener}]'
echo $BUCKET
```

#### Chạy quét

2. Quét Amazon S3 và Amazon EC2 ở Singapore, chỉ với tài nguyên có tag `Project=fcj-screener`:

```bash
screener --regions ap-southeast-1 --services s3,ec2 \
  --tags Project=fcj-screener --disable-custom-pages 1 --beta 1
```

3. Lần quét này mất khoảng 30 giây. Cuối cùng, Service Screener in ra đường dẫn báo cáo `/tmp/service-screener-v2/output.zip` và dòng `Scan Completed`.

![Quét xong](/images/3/1-scan-done.png)

{{% notice info %}}
Các check áp dụng cho cả Region hoặc cả tài khoản, như **EBS encryption by default**, không gắn với tài nguyên có tag, nên luôn xuất hiện trong báo cáo.
{{% /notice %}}

{{% notice warning %}}
Kể cả khi không dùng tuỳ chọn Well-Architected ở [chương 5](../5-well-architected-integration/), phiên bản hiện tại vẫn ghi kết quả vào một workload trong Well-Architected Tool tên **`SS_Report`** ở Region được quét, và tự tạo workload này nếu chưa có. Mỗi lần chạy cũng để lại một stack CloudFormation rỗng tên `ssv2-...`. Hãy xoá cả hai ở chương [Dọn dẹp tài nguyên](../7-clean-up/).
{{% /notice %}}

#### Một số tuỳ chọn hữu ích

| Tuỳ chọn | Ví dụ | Tác dụng |
|---|---|---|
| `--regions` | `--regions ap-southeast-1,us-east-1` hoặc `--regions ALL` | Các Region cần quét (bắt buộc) |
| `--services` | `--services s3,ec2,rds,iam` | Chỉ quét các dịch vụ này. Nếu bỏ trống, quét mọi dịch vụ được hỗ trợ |
| `--tags` | `--tags env=prod%department=hr,coe` | Chỉ quét tài nguyên có các tag này. `%` ngăn cách các key, `,` ngăn cách các value |
| `--beta 1` | | Dùng báo cáo Cloudscape mới (cần bước build ở chương 2) |
| `--disable-custom-pages 1` | | Bỏ qua các trang Cost Optimization Hub, Trusted Advisor và finding liên dịch vụ. Quét nhanh hơn; các trang này áp dụng cho cả tài khoản, không lọc theo tag |
| `--suppress_file` | `--suppress_file ./suppressions.json` | Ẩn các finding bạn đã chấp nhận |
| `--resume` | `--resume 1` hoặc `--resume 0` | Tiếp tục lần quét bị gián đoạn, hoặc buộc quét lại từ đầu |
| `--sequential 1` | | Chạy các check lần lượt thay vì song song |

Nếu một lần quét cùng Region và dịch vụ bị gián đoạn, Service Screener sẽ hỏi `Resume from where it left off? [y/n]`. Trả lời `y` để tiếp tục hoặc `n` để quét lại từ đầu.
