---
title : "Cấu hình WordPress"
weight : 2
chapter : false
pre : " <b> 3.2 </b> "
---

Bây giờ bạn kết nối WordPress với cơ sở dữ liệu. Script cấu hình chạy **trên instance**: nó đọc thông tin đăng nhập cơ sở dữ liệu từ AWS Secrets Manager qua interface endpoint và ghi vào `wp-config.php`. Mật khẩu không bao giờ xuất hiện trên CloudShell, trên console hay trong hướng dẫn này.

File `wp-config.php` được tạo ra:
- Tạo các security key và salt ngay trên máy bằng `random_bytes` (instance không gọi được `api.wordpress.org`).
- Dùng `MYSQL_CLIENT_FLAGS = MYSQLI_CLIENT_SSL`, vì RDS for MySQL 8.4 mặc định yêu cầu kết nối mã hoá.
- Tạo `WP_HOME` và `WP_SITEURL` từ host của request, nhờ đó cùng một AMI chạy được phía sau load balancer và phía sau CloudFront, đồng thời coi request là HTTPS khi CloudFront báo người dùng đang dùng HTTPS.
- Chặn các request HTTP đi ra ngoài và chặn cập nhật tự động, vì các chức năng này không hoạt động khi không có internet.

1. Trong **AWS CloudShell**, tạo hai file dưới đây.

   `gen.php` – tạo file `wp-config.php`:

   ```php
   <?php
   [$_, $h, $u, $p] = $argv;
   $salts = '';
   foreach (['AUTH_KEY','SECURE_AUTH_KEY','LOGGED_IN_KEY','NONCE_KEY','AUTH_SALT','SECURE_AUTH_SALT','LOGGED_IN_SALT','NONCE_SALT'] as $k) {
     $salts .= "define('$k', '" . bin2hex(random_bytes(32)) . "');\n";
   }
   $e = fn($v) => var_export($v, true);
   echo <<<PHP
   <?php
   define('DB_NAME', 'wordpress');
   define('DB_USER', {$e($u)});
   define('DB_PASSWORD', {$e($p)});
   define('DB_HOST', {$e($h)});
   define('DB_CHARSET', 'utf8mb4');
   define('DB_COLLATE', '');
   define('MYSQL_CLIENT_FLAGS', MYSQLI_CLIENT_SSL);
   $salts
   \$table_prefix = 'wp_';
   if ((\$_SERVER['HTTP_CLOUDFRONT_FORWARDED_PROTO'] ?? '') === 'https' || (\$_SERVER['HTTP_X_FORWARDED_PROTO'] ?? '') === 'https') {
     \$_SERVER['HTTPS'] = 'on';
   }
   if (!empty(\$_SERVER['HTTP_HOST'])) {
     define('WP_HOME', ((\$_SERVER['HTTPS'] ?? '') === 'on' ? 'https' : 'http') . '://' . \$_SERVER['HTTP_HOST']);
     define('WP_SITEURL', WP_HOME);
   }
   define('WP_HTTP_BLOCK_EXTERNAL', true);
   define('AUTOMATIC_UPDATER_DISABLED', true);
   define('DISALLOW_FILE_EDIT', true);
   define('WP_DEBUG', false);
   if (!defined('ABSPATH')) define('ABSPATH', __DIR__ . '/');
   require_once ABSPATH . 'wp-settings.php';

   PHP;
   ```

   `config.sh` – chạy trên instance:

   ```bash
   DBHOST=$1; SECRET=$2; B=fcj-wordpress-files-1795416336
   aws s3 cp s3://$B/gen.php /root/gen.php --region ap-northeast-1 --no-progress
   J=$(aws secretsmanager get-secret-value --secret-id "$SECRET" --region ap-northeast-1 --query SecretString --output text)
   php /root/gen.php "$DBHOST" "$(echo "$J" | jq -r .username)" "$(echo "$J" | jq -r .password)" > /var/www/html/wp-config.php
   chown apache:apache /var/www/html/wp-config.php; chmod 640 /var/www/html/wp-config.php
   ADMINPW=$(openssl rand -base64 18)
   echo "$ADMINPW" > /root/wp-admin-password.txt; chmod 600 /root/wp-admin-password.txt
   cd /var/www/html
   wp --allow-root core install --url=http://localhost --title='FCJ WordPress on AWS' --admin_user=fcjadmin --admin_password="$ADMINPW" --admin_email=admin@example.com --skip-email
   wp --allow-root option get blogname
   wp --allow-root db query 'SELECT VERSION();'
   ```

2. Upload các file lên bucket của bạn:

   ```bash
   aws s3 cp gen.php s3://fcj-wordpress-files-1795416336/
   aws s3 cp config.sh s3://fcj-wordpress-files-1795416336/
   ```

3. Khi `wordpress-db` đã có endpoint (trạng thái có thể vẫn là **Modifying**), chạy script trên instance bằng **Run Command**:

   ```bash
   IID=$(aws ec2 describe-instances --filters Name=tag:Name,Values=wordpress-setup Name=instance-state-name,Values=running --query "Reservations[0].Instances[0].InstanceId" --output text)
   read DBHOST SECRET <<< $(aws rds describe-db-instances --db-instance-identifier wordpress-db --query "DBInstances[0].[Endpoint.Address,MasterUserSecret.SecretArn]" --output text)
   CID=$(aws ssm send-command --instance-ids $IID --document-name AWS-RunShellScript \
     --parameters "commands=['aws s3 cp s3://fcj-wordpress-files-1795416336/config.sh /root/config.sh --region ap-northeast-1 --no-progress && bash /root/config.sh $DBHOST $SECRET']" \
     --query Command.CommandId --output text)
   sleep 25
   aws ssm get-command-invocation --command-id $CID --instance-id $IID --query "[Status,StandardOutputContent,StandardErrorContent]" --output text | tail -12
   ```

4. Kết quả hiển thị (1) `Success: WordPress installed successfully.`, tiêu đề website và phiên bản MySQL `8.4.x` do cơ sở dữ liệu trả về qua kết nối TLS.

   ![WordPress đã được cài đặt](/images/3-install/3.2-wp-installed.png)

{% notice info %}
Script tạo tài khoản quản trị WordPress `fcjadmin` với mật khẩu ngẫu nhiên và lưu vào `/root/wp-admin-password.txt` trên instance. Để xem mật khẩu, kết nối bằng Session Manager và chạy `sudo cat /root/wp-admin-password.txt`.
{% /notice %}

{% notice warning %}
WordPress được cài đặt trước khi có load balancer, vì vậy trang cài đặt `wp-admin/install.php` không bao giờ bị lộ ra internet.
{% /notice %}
