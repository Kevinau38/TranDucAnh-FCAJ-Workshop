---
title : "Configure WordPress"
weight : 3
chapter : false
pre : " <b> 2.3. </b> "
---

You now point WordPress at the database and run the installation. You do this from **AWS CloudShell**, a browser terminal that already has the AWS CLI and your permissions. The commands read the database password from Lightsail and pass it to the instance through SSH, so the password never appears on screen or in a file.

1. Wait until the database `fcj-lightsail-db` shows **Available**.

2. At the bottom left of the Lightsail console, choose **CloudShell**. Check that the tab shows the Region **ap-southeast-1**.

3. Create a small helper script. `lsssh.sh` asks Lightsail for a temporary SSH key and certificate for an instance (valid for a short time) and opens an SSH session with them, so you do not need to download a key pair.

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

4. Check that the launch script has finished. The output must show `status: done` and the WP-CLI version.

```bash
./lsssh.sh fcj-wordpress 'cloud-init status --wait; wp --version'
```

5. Read the database endpoint, the user name and the static IP into variables, and generate a password for the WordPress administrator. **Save the printed admin password**: you need it to log in to WordPress.

```bash
DB=fcj-lightsail-db
EP=$(aws lightsail get-relational-database --relational-database-name $DB --query relationalDatabase.masterEndpoint.address --output text)
MU=$(aws lightsail get-relational-database --relational-database-name $DB --query relationalDatabase.masterUsername --output text)
WPIP=$(aws lightsail get-static-ip --static-ip-name fcj-wordpress-ip --query staticIp.ipAddress --output text)
ADMIN_PASS=$(openssl rand -base64 18)
echo "WordPress admin password: $ADMIN_PASS"
```

6. Configure `wp-config.php` and install WordPress. The first command in the pipe reads the managed database password; the instance reads it from standard input.

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

The output ends with `Success: WordPress installed successfully.` and `Success: Database checked.`

![WordPress installed](/images/2/9-cloudshell.png)

7. In your browser, open `http://<static-ip>/`. The blog **FCJ Lightsail Blog** is running. To manage it, open `http://<static-ip>/wp-admin/` and log in as `fcjadmin` with the password from step 5.

![WordPress blog](/images/2/10-wordpress.png)

{{% notice tip %}}
This lab uses plain HTTP to stay short. For a real site, point a domain name at the static IP (Lightsail **Domains & DNS**) and add a free TLS certificate, for example with Certbot or a Lightsail distribution.
{{% /notice %}}
