---
title : "Kết nối và cài đặt web server"
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---

Bạn kết nối vào instance private bằng **Session Manager** rồi cài Apache, PHP, WordPress và WP-CLI. Mọi package đều lấy từ kho Amazon Linux trên Amazon S3, còn WordPress lấy từ bucket của bạn, vì vậy instance không bao giờ cần kết nối internet.

1. Mở [Amazon EC2 console](https://console.aws.amazon.com/ec2/) → **Instances**, chọn `wordpress-setup` rồi chọn **Connect** → **Session Manager** → **Connect**.

2. Một terminal mở ra trên trình duyệt với user `ssm-user`. Chuyển sang `root`:

   ```bash
   sudo -i
   ```

3. Chạy các câu lệnh sau. Thay tên bucket bằng bucket của bạn:

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

   - Apache phục vụ website từ thư mục `/var/www/html` và chuyển các file PHP cho `php-fpm` xử lý.
   - `health.html` là một trang tĩnh nhỏ mà load balancer dùng để kiểm tra sức khoẻ (health check) ở phần 4.

4. Kiểm tra kết quả:

   ```bash
   hostname -I; php -v | head -1; wp --allow-root --version; systemctl is-active httpd php-fpm; curl -s localhost/health.html; ls /var/www/html | head -5
   curl -s -m 5 -o /dev/null -w 'internet: %{http_code}\n' https://wordpress.org || echo 'internet: blocked (no NAT, private subnet)'
   ```

   - (1) Instance chỉ có địa chỉ IP private, PHP và WP-CLI đã được cài đặt.
   - (2) Request đến `wordpress.org` thất bại: instance thật sự không có kết nối internet.

   ![Kiểm tra instance](/images/3-install/3.1-ssm-verify.png)

{% notice note %}
Ảnh chụp terminal được hiển thị với nền sáng để dễ đọc. Terminal Session Manager của bạn có nền tối.
{% /notice %}
