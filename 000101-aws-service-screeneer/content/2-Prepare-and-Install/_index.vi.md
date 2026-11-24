---
title : "Chuẩn bị và cài đặt"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

#### Quyền cần có

User hoặc role chạy Service Screener cần:

| Quyền | Để làm gì |
|---|---|
| `ReadOnlyAccess` (AWS managed policy) | Đọc cấu hình tài nguyên |
| `AWSCloudShellFullAccess` (AWS managed policy) | Dùng CloudShell |
| `cloudformation:CreateStack`, `cloudformation:DeleteStack` | Mỗi lần chạy, Service Screener tạo một stack rỗng tên `ssv2-...` làm dấu vết kiểm toán |
| Quyền ghi vào Well-Architected Tool, ví dụ `WellArchitectedConsoleFullAccess` | Chỉ cần cho [chương 5](../5-well-architected-integration/) và workload mặc định nói ở chương 3 |
| `s3:CreateBucket`, `s3:PutBucketTagging`, `s3:DeleteBucket` | Chỉ để tạo và xoá bucket demo |

#### Mở CloudShell

1. Đăng nhập AWS console và chuyển sang Region **Asia Pacific (Singapore)**.
2. Chọn biểu tượng **CloudShell** trên thanh trên cùng, hoặc mở trực tiếp [CloudShell](https://console.aws.amazon.com/cloudshell/home?region=ap-southeast-1). Chờ đến khi dấu nhắc lệnh xuất hiện.

![CloudShell](/images/2/1-cloudshell.png)

#### Cài Service Screener

3. Service Screener cần Python 3.13. Cài đặt:

```bash
sudo yum install python3.13 -y
python3.13 --version
```

4. Tạo môi trường ảo trong `/tmp`, tải công cụ và cài các thư viện cần thiết. Lệnh chạy xong không báo lỗi.

```bash
cd /tmp
python3.13 -m venv .
source bin/activate
python3.13 -m pip install --upgrade pip
rm -rf service-screener-v2
git clone https://github.com/aws-samples/service-screener-v2.git
cd service-screener-v2
pip install -r requirements.txt
python3.13 scripts/unzip_botocore_lambda_runtime.py
```

![Cài đặt xong](/images/2/2-install.png)

{{% notice note %}}
`pip` có thể in cảnh báo `BuildBackendWarning` về editable installation. Bạn có thể bỏ qua.
{{% /notice %}}

5. Build giao diện báo cáo mới (**Cloudscape UI**), rồi tạo alias `screener`. Bước build mất chưa tới một phút và kết thúc bằng `built in ...`.

```bash
cd cloudscape-ui
npm install
npm run build
cd ..
alias screener='python3 $(pwd)/main.py'
```

![Build Cloudscape UI xong](/images/2/3-build.png)

{{% notice warning %}}
CloudShell xoá thư mục `/tmp` khi phiên làm việc kết thúc, và alias chỉ tồn tại trong terminal hiện tại. Nếu quay lại sau hoặc mở tab mới, hãy chạy lại bước 3–5.
{{% /notice %}}
