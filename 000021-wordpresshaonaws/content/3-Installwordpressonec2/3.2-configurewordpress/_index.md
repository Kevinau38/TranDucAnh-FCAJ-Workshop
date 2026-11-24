---
title : "Configure WordPress"
weight : 2
chapter : false
pre : " <b> 3.2 </b> "
---

Now you connect WordPress to the database. The configuration script runs **on the instance**: it reads the database credentials from AWS Secrets Manager through the interface endpoint and writes them into `wp-config.php`. The password never appears in CloudShell, in the console or in this guide.

The generated `wp-config.php`:
- Creates the security keys and salts locally with `random_bytes` (the instance cannot call `api.wordpress.org`).
- Uses `MYSQL_CLIENT_FLAGS = MYSQLI_CLIENT_SSL`, because RDS for MySQL 8.4 requires encrypted connections by default.
- Builds `WP_HOME` and `WP_SITEURL` from the request host, so the same AMI works behind the load balancer and behind CloudFront, and treats requests as HTTPS when CloudFront says the viewer used HTTPS.
- Blocks outgoing HTTP requests and automatic updates, which cannot work without internet access.

1. In **AWS CloudShell**, create the two files below.

   `gen.php` – builds `wp-config.php`:

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

   `config.sh` – runs on the instance:

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

2. Upload the files to your bucket:

   ```bash
   aws s3 cp gen.php s3://fcj-wordpress-files-1795416336/
   aws s3 cp config.sh s3://fcj-wordpress-files-1795416336/
   ```

3. When `wordpress-db` has an endpoint (the status can still be **Modifying**), run the script on the instance with **Run Command**:

   ```bash
   IID=$(aws ec2 describe-instances --filters Name=tag:Name,Values=wordpress-setup Name=instance-state-name,Values=running --query "Reservations[0].Instances[0].InstanceId" --output text)
   read DBHOST SECRET <<< $(aws rds describe-db-instances --db-instance-identifier wordpress-db --query "DBInstances[0].[Endpoint.Address,MasterUserSecret.SecretArn]" --output text)
   CID=$(aws ssm send-command --instance-ids $IID --document-name AWS-RunShellScript \
     --parameters "commands=['aws s3 cp s3://fcj-wordpress-files-1795416336/config.sh /root/config.sh --region ap-northeast-1 --no-progress && bash /root/config.sh $DBHOST $SECRET']" \
     --query Command.CommandId --output text)
   sleep 25
   aws ssm get-command-invocation --command-id $CID --instance-id $IID --query "[Status,StandardOutputContent,StandardErrorContent]" --output text | tail -12
   ```

4. The output shows (1) `Success: WordPress installed successfully.`, the site title, and the MySQL version `8.4.x` returned by the database over TLS.

   ![WordPress installed](/images/3-install/3.2-wp-installed.png)

{% notice info %}
The script creates the WordPress administrator `fcjadmin` with a random password and saves it in `/root/wp-admin-password.txt` on the instance. To read it, connect with Session Manager and run `sudo cat /root/wp-admin-password.txt`.
{% /notice %}

{% notice warning %}
WordPress is installed before the load balancer exists, so the installation page `wp-admin/install.php` is never reachable from the internet.
{% /notice %}
