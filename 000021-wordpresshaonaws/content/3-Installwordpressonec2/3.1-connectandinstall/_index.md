---
title : "Connect and install the web server"
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---

You connect to the private instance with **Session Manager** and install Apache, PHP, WordPress and WP-CLI. All packages come from the Amazon Linux repositories in Amazon S3, and WordPress comes from your own bucket, so the instance never needs internet access.

1. Open the [Amazon EC2 console](https://console.aws.amazon.com/ec2/) → **Instances**, select `wordpress-setup` and choose **Connect** → **Session Manager** → **Connect**.

2. A terminal opens in your browser as `ssm-user`. Switch to `root`:

   ```bash
   sudo -i
   ```

3. Run the following commands. Replace the bucket name with your own:

   ```bash
   B=fcj-wordpress-files-1795416336
   dnf install -y -q httpd php php-fpm php-mysqlnd php-gd php-mbstring php-xml php-intl jq mariadb105
   aws s3 cp s3://$B/latest.tar.gz /tmp/wp.tgz --region ap-northeast-1 --no-progress
   aws s3 cp s3://$B/wp-cli.phar /usr/local/bin/wp --region ap-northeast-1 --no-progress
   chmod +x /usr/local/bin/wp
   tar -xzf /tmp/wp.tgz -C /var/www/html --strip-components=1
   echo OK > /var/www/html/health.html
   chown -R apache:apache /var/www/html
   systemctl enable --now php-fpm httpd
   php -v | head -1; wp --allow-root --version; curl -s localhost/health.html
   ```

   - Apache serves the site from `/var/www/html` and passes PHP files to `php-fpm`.
   - `health.html` is a small static page that the load balancer uses for health checks in part 4.

4. Check the result:

   ```bash
   hostname -I; php -v | head -1; wp --allow-root --version; systemctl is-active httpd php-fpm; curl -s localhost/health.html; ls /var/www/html | head -5
   curl -s -m 5 -o /dev/null -w 'internet: %{http_code}\n' https://wordpress.org || echo 'internet: blocked (no NAT, private subnet)'
   ```

   - (1) The instance has only a private IP address, and PHP and WP-CLI are installed.
   - (2) The request to `wordpress.org` fails: the instance really has no internet access.

   ![Verify the instance](/images/3-install/3.1-ssm-verify.png)

{% notice note %}
The terminal screenshots are shown with a light background to make them easier to read. Your Session Manager terminal has a dark background.
{% /notice %}
