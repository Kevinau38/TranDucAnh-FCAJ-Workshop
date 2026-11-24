---
title : "Triển khai instance"
weight : 1
chapter : false
pre : " <b> 2.1. </b> "
---

1. Trong Lightsail console, chọn **Instances**, rồi **Create instance**. Đảm bảo vị trí là **Singapore, Zone A (ap-southeast-1a)**, cùng zone với database.

2. Ở mục **Select a platform**, chọn **Linux operating system**. Ở mục **Select a blueprint**, chọn **Ubuntu 24.04 LTS**.

![Blueprint Ubuntu](/images/2/1-blueprint.png)

3. Ở mục **Advanced settings**, dán script sau vào **Launch script**. Script chạy một lần với quyền root khi instance khởi động lần đầu. Nó cài Apache, PHP 8.3 và MySQL client từ kho Ubuntu, tải bản WordPress mới nhất vào `/var/www/html` và cài **WP-CLI**, công cụ dòng lệnh của WordPress.

```bash
#!/bin/bash
# Install Apache, PHP and the MySQL client (Ubuntu 24.04)
apt-get update
DEBIAN_FRONTEND=noninteractive apt-get install -y apache2 php libapache2-mod-php php-mysql php-gd php-xml php-mbstring php-curl php-zip php-intl mysql-client
# Download WordPress into the web root
cd /var/www/html
curl -sSO https://wordpress.org/latest.tar.gz
tar xzf latest.tar.gz --strip-components=1
rm -f latest.tar.gz index.html
cp wp-config-sample.php wp-config.php
chown -R www-data:www-data /var/www/html
# Install WP-CLI
curl -sSo /usr/local/bin/wp https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
chmod +x /usr/local/bin/wp
systemctl restart apache2
```

![Launch script](/images/2/2-launch-script.png)

4. Ở mục **Select a size**, chọn gói **7 USD** (1 GB bộ nhớ, 2 vCPU, 40 GB SSD).

![Gói instance](/images/2/3-plan.png)

5. Ở mục **Instance name**, nhập `fcj-wordpress`. Ở mục **Tagging options**, thêm tag `Project` = `fcj-lightsail`. Chọn **Create instance**.

![Tên và tag](/images/2/4-name-tag.png)

6. Instance khởi động trong khoảng một phút. Launch script cần thêm khoảng 3 phút để cài xong các gói.

![Instance đang khởi động](/images/2/5-starting.png)
