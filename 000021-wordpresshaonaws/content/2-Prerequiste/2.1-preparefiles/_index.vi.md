---
title : "Chuẩn bị file cài đặt trên Amazon S3"
weight : 1
chapter : false
pre : " <b> 2.1 </b> "
---

Máy chủ web chạy trong private subnet và không có NAT gateway, vì vậy chúng không thể tải WordPress từ internet. Ở bước này, bạn tải file một lần trong **AWS CloudShell** và lưu vào một bucket S3. Sau đó các instance sẽ đọc bucket này qua S3 gateway endpoint.

1. Mở [AWS Management Console](https://console.aws.amazon.com/), kiểm tra Region đang là **Asia Pacific (Tokyo)**, rồi chọn biểu tượng **CloudShell** ở góc dưới bên trái.

2. Chạy câu lệnh sau. Câu lệnh tạo một bucket có tên duy nhất, tải gói WordPress mới nhất và WP-CLI, rồi upload cả hai file lên bucket:

   ```bash
   BUCKET=fcj-wordpress-files-$(date +%s) && aws s3 mb s3://$BUCKET && curl -sSLO https://wordpress.org/latest.tar.gz && curl -sSLO https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar && aws s3 cp latest.tar.gz s3://$BUCKET/ --no-progress && aws s3 cp wp-cli.phar s3://$BUCKET/ --no-progress && aws s3 ls s3://$BUCKET/ && echo "BUCKET=$BUCKET"
   ```

3. Kiểm tra bucket đã có `latest.tar.gz` và `wp-cli.phar`, rồi ghi lại tên bucket ở dòng cuối cùng. Bạn sẽ dùng tên này ở các bước sau.

   ![Upload file lên S3](/images/2-prerequisite/2.1-s3-files.png)

{{% notice note %}}
Ở các bước còn lại của bài lab, hãy thay `fcj-wordpress-files-1795416336` bằng tên bucket của bạn.
{{% /notice %}}
