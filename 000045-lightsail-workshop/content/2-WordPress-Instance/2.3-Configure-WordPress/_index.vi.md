---
title : "Cấu hình WordPress"
weight : 3
chapter : false
pre : " <b> 2.3. </b> "
---

Bây giờ bạn trỏ WordPress tới database và chạy bước cài đặt. Bạn làm việc này từ **AWS CloudShell**, một terminal trên trình duyệt đã có sẵn AWS CLI và quyền của bạn. Các lệnh đọc mật khẩu database từ Lightsail rồi chuyển cho instance qua SSH, nên mật khẩu không bao giờ hiện trên màn hình hay nằm trong file.

1. Chờ đến khi database `fcj-lightsail-db` ở trạng thái **Available**.

2. Ở góc dưới bên trái Lightsail console, chọn **CloudShell**. Kiểm tra tab đang ở Region **ap-southeast-1**.

3. Tạo một script hỗ trợ nhỏ. `lsssh.sh` xin Lightsail một SSH key và certificate tạm thời (chỉ có hiệu lực trong thời gian ngắn) cho instance rồi mở phiên SSH bằng chúng, nên bạn không cần tải key pair về.

```bash
cat > lsssh.sh <<'EOF'
#!/bin/bash
# Usage: ./lsssh.sh <instance-name> [command]
I=$1; shift
aws lightsail get-instance-access-details --instance-name "$I" --protocol ssh --output json > /tmp/ad.json
jq -r .accessDetails.privateKey /tmp/ad.json > /tmp/lskey && chmod 600 /tmp/lskey
jq -r .accessDetails.certKey /tmp/ad.json > /tmp/lskey-cert.pub
U=$(jq -r .accessDetails.username /tmp/ad.json); IP=$(jq -r .accessDetails.ipAddress /tmp/ad.json)
ssh -i /tmp/lskey -o CertificateFile=/tmp/lskey-cert.pub -o StrictHostKeyChecking=no -o LogLevel=ERROR "$U@$IP" "$@"
EOF
chmod +x lsssh.sh
```

4. Kiểm tra launch script đã chạy xong. Kết quả phải có `status: done` và phiên bản WP-CLI.

```bash
./lsssh.sh fcj-wordpress 'cloud-init status --wait; wp --version'
```

5. Đọc endpoint database, user name và static IP vào biến, đồng thời tạo mật khẩu cho tài khoản quản trị WordPress. **Lưu lại mật khẩu admin được in ra**: bạn cần nó để đăng nhập WordPress.

```bash
DB=fcj-lightsail-db
EP=$(aws lightsail get-relational-database --relational-database-name $DB --query relationalDatabase.masterEndpoint.address --output text)
MU=$(aws lightsail get-relational-database --relational-database-name $DB --query relationalDatabase.masterUsername --output text)
WPIP=$(aws lightsail get-static-ip --static-ip-name fcj-wordpress-ip --query staticIp.ipAddress --output text)
ADMIN_PASS=$(openssl rand -base64 18)
echo "WordPress admin password: $ADMIN_PASS"
```

6. Cấu hình `wp-config.php` và cài WordPress. Lệnh đầu tiên trong pipe đọc mật khẩu database được quản lý; instance nhận mật khẩu qua standard input.

```bash
{ aws lightsail get-relational-database-master-user-password --relational-database-name $DB --query masterUserPassword --output text; echo "$ADMIN_PASS"; } | \
./lsssh.sh fcj-wordpress "read -r DBP; read -r ADP; cd /var/www/html && \
 sudo -u www-data wp config set DB_HOST $EP --quiet && \
 sudo -u www-data wp config set DB_USER $MU --quiet && \
 sudo -u www-data wp config set DB_NAME dbwordpress --quiet && \
 sudo -u www-data wp config set DB_PASSWORD \"\$DBP\" --quiet && \
 sudo -u www-data wp config shuffle-salts --quiet && \
 sudo -u www-data wp core install --url=http://$WPIP --title='FCJ Lightsail Blog' \
   --admin_user=fcjadmin --admin_password=\"\$ADP\" --admin_email=admin@example.com --skip-email && \
 sudo -u www-data wp db check | tail -2" 2>&1 | grep -v Warning
```

Kết quả kết thúc bằng `Success: WordPress installed successfully.` và `Success: Database checked.`

![WordPress đã cài xong](/images/2/9-cloudshell.png)

7. Trên trình duyệt, mở `http://<static-ip>/`. Blog **FCJ Lightsail Blog** đã chạy. Để quản trị, mở `http://<static-ip>/wp-admin/` và đăng nhập bằng `fcjadmin` với mật khẩu ở bước 5.

![Blog WordPress](/images/2/10-wordpress.png)

{{% notice tip %}}
Bài lab dùng HTTP cho ngắn gọn. Với website thật, hãy trỏ tên miền tới static IP (mục **Domains & DNS** của Lightsail) và thêm chứng chỉ TLS miễn phí, ví dụ bằng Certbot hoặc Lightsail distribution.
{{% /notice %}}
