---
title : "Deploy the instance"
weight : 1
chapter : false
pre : " <b> 2.1. </b> "
---

1. In the Lightsail console, choose **Instances**, then **Create instance**. Make sure the location is **Singapore, Zone A (ap-southeast-1a)**, the same zone as the database.

2. Under **Select a platform**, choose **Linux operating system**. Under **Select a blueprint**, choose **Ubuntu 24.04 LTS**.

![Ubuntu blueprint](/images/2/1-blueprint.png)

3. Under **Advanced settings**, paste the following script into **Launch script**. It runs once, as root, the first time the instance starts. It installs Apache, PHP 8.3 and the MySQL client from the Ubuntu repositories, downloads the latest WordPress into `/var/www/html`, and installs **WP-CLI**, the WordPress command-line tool.

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

4. Under **Select a size**, choose the **USD 7** plan (1 GB memory, 2 vCPUs, 40 GB SSD).

![Instance plan](/images/2/3-plan.png)

5. Under **Instance name**, enter `fcj-wordpress`. Under **Tagging options**, add the tag `Project` = `fcj-lightsail`. Choose **Create instance**.

![Name and tag](/images/2/4-name-tag.png)

6. The instance starts in a minute. The launch script needs about 3 more minutes to finish installing the packages.

![Instance starting](/images/2/5-starting.png)
