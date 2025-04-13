# Apache2 Setup on Debian

This guide outlines how to install and configure Apache2, PHP, MySQL, SSL, phpMyAdmin, and WordPress on a Debian-based system.

### System Update

Before beginning, make sure the system is up-to-date.

```
apt update
```

```
apt upgrade
```

### Installing Apache2

Install the Apache web server and basic networking tools:
```
apt install apache2
```

```
apt install net-tools
```

Check running services and confirm Apache installation:
```
netstat -nltup
```

```
dpkg -l | grep apache
```

Edit Apache configuration file:
```
vim /etc/apache2/apache2.conf
```

Restart and enable Apache service:
```
systemctl restart apache2.service
```

```
systemctl enable apache2.service
```

Check network again to ensure Apache is listening:
```
netstat -nltup
```
### Disable Directory Listing

To improve security, disable Apache's directory listing feature:
```
sed -i "s/Options Indexes FollowSymLinks/Options FollowSymLinks/" /etc/apache2/apache2.conf
```

```
systemctl restart apache2.service
```

### Installing PHP

Search for available PHP versions:
```
apt search php | grep "php/stable"
```

Install PHP and commonly used extensions:
```
apt install php php8.2 php8.2-common php8.2-mbstring php8.2-xmlrpc php8.2-soap php8.2-gd php8.2-xml php8.2-intl php8.2-mysql php8.2-cli php8.2-ldap php8.2-zip php8.2-curl php-xml composer
```

Configure PHP settings:
```
vim /etc/php/8.2/apache2/php.ini
```

Example recommended values:
```
memory_limit = 512M
max_execution_time = 500
max_input_vars = 10000
upload_max_filesize = 2048M
post_max_size = 2048M
allow_url_fopen = On
```
Insert the following:
```
<?php
    phpinfo();
?>
```

Set correct ownership:
```
chown -Rv www-data:www-data /var/www/html/phpinfo.php
```
### Installing MySQL Server

Install prerequisites:
```
apt install gnupg
```

```
apt install wget
```

Download MySQL APT configuration package:
```
wget https://dev.mysql.com/get/mysql-apt-config_0.8.24-1_all.deb
```

Install the package:
```
apt install ./mysql-apt-config_0.8.24-1_all.deb
```

Update package lists:
```
apt update
```

Install MySQL server:
```
apt install mysql-community-server
```

Enable and start MySQL:
```
systemctl restart mysql.service
```

```
systemctl enable mysql.service
```
Check open ports:
```
netstat -nltup
```

Run secure installation:
```
mysql_secure_installation
```

Access MySQL:
```
mysql -u root -p
```

Inside MySQL:
```
show databases;
```

Create remote root user (optional for remote access):
```
CREATE USER 'root'@'%' IDENTIFIED WITH caching_sha2_password BY '***';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
```
### Enabling SSL on Apache

Enable SSL module and default SSL site:
```
a2enmod ssl
```

```
systemctl restart apache2
```

```
a2ensite default-ssl
```

```
systemctl reload apache2
```

Check network again:
```
netstat -nltup
```

```
service apache2 reload
```
### phpMyAdmin Installation

Download and extract phpMyAdmin:
```
wget https://files.phpmyadmin.net/phpMyAdmin/5.2.0/phpMyAdmin-5.2.0-all-languages.zip
```

```
unzip phpMyAdmin-5.2.0-all-languages.zip
```

Move to web directory:
```
mv -v phpMyAdmin-5.2.0-all-languages /var/www/html/phpmyadmin
```

```
cd /var/www/html
```

Set ownership:
```
chown -Rv www-data:www-data /var/www/html/phpmyadmin
```

Create config file:
```
cp -v /var/www/html/phpmyadmin/config.sample.inc.php /var/www/html/phpmyadmin/config.inc.php
```

```
chown -Rv www-data:www-data config.inc.php
```

Generate a blowfish secret:
```
pwgen 32 -1
```
Edit the config file:
```
vim /var/www/html/phpmyadmin/config.inc.php
```

Insert:
```
$cfg['blowfish_secret'] = 'ophixah6ufboshae9veipahK4daeqah';
```

MySQL remote login example:
```
mysql -h 192.168.1.23 -u root -p
```
### Creating a Self-Signed SSL Certificate
```
mkdir /etc/apache2/ssl
```

Generate a certificate:
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/armour.local.key -out /etc/apache2/ssl/armour.local.crt
```

View certificates:
```
cat /etc/apache2/ssl/armour.local.key
```

```
cat /etc/apache2/ssl/armour.local.crt
```

Update hosts file:
```
vim /etc/hosts
```

Example:
```
192.168.1.45    armour.local www.armour.local
```

### Configure Apache Virtual Host for HTTPS

Create the site directory:
```
mkdir -p /var/www/html/armour.local
```

Create the SSL config:
```
vim /etc/apache2/sites-available/armour.local-ssl.conf
```

Example config:
```
<VirtualHost *:443>
    ServerAdmin webmaster@armour.local
    ServerName armour.local
    ServerAlias www.armour.local
    DocumentRoot /var/www/html/armour.local
    SSLEngine on
    SSLCertificateFile /etc/apache2/ssl/armour.local.crt
    SSLCertificateKeyFile /etc/apache2/ssl/armour.local.key
    <Directory /var/www/html/armour.local>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/armour.local-error.log
    CustomLog ${APACHE_LOG_DIR}/armour.local-access.log combined
</VirtualHost>
```

Enable and restart:
```
a2ensite armour.local-ssl.conf
```
### Install WordPress

Download and extract:
```
wget https://wordpress.org/latest.zip
```

```
unzip latest.zip
```

Move to appropriate directory:
```
mv -v wordpress/* /var/www/html/armour.local
```

Set permissions:
```
chown -Rv www-data:www-data /var/www/html/armour.local
```

### Virtual Host Binding and Multi-site Setup

Create directories:
```
mkdir /var/www/html/site1
mkdir /var/www/html/site2
mkdir /var/www/html/site3
```

Set ownership:
```
chown -R www-data:www-data /var/www/html/site1/
chown -Rv www-data:www-data /var/www/html/site*
```

Configure each site's .conf files (site1, site2, site3):
```
<VirtualHost 192.168.1.25:80>
    ServerName armour.local
    DocumentRoot /var/www/html/site1/
    <Directory /var/www/html/site1/>
        Options FollowSymLinks
        AllowOverride All
        Order allow,deny
        allow from all
    </Directory>
    ErrorLog /var/log/apache2/site1-error.log
    CustomLog /var/log/apache2/site1-access.log common
    ServerAlias www.armour.local
</VirtualHost>
```

Enable sites and modules:
```
a2dissite 000-default
a2dissite default-ssl
a2ensite site1.conf
a2enmod rewrite
systemctl restart apache2
```

List enabled/available sites:
```
ls -lh /etc/apache2/sites-enabled
ls -lh /etc/apache2/sites-available
```
 