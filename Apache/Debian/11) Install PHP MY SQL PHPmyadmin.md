# PHP-and-Mysql-Installation-and-Configuration
## PHP Installation and Configuration

### Add PHP Repository

```
apt update
apt install software-properties-common apt-transport-https lsb-release ca-certificates
```

Add the PHP repository to get newer PHP versions:

```
add-apt-repository ppa:ondrej/php
apt update
```

### Search for Available PHP Versions
```
apt search php | grep -E "^php[0-9]+"
```
```
apt show php8.1
```
```
apt show php8.2
```

### Install PHP and Required Extensions
```
apt install php php-common php-cli php-opcache php-gd php-curl php-mysql php-xml php-mbstring php-pear php-intl php-zip php-fpm
```

### Verify PHP Installation
```
php -v
```

### Enable PHP for Apache
```
a2enmod php
```

### Restart Apache
```
systemctl restart apache2.service
```

### Create phpinfo.php for Testing
```
vim /var/www/html/phpinfo.php
```

Add the following content:
```php
<?php
    phpinfo();
?>
```

Access `phpinfo.php` via:
http://192.168.1.32/phpinfo.php

### MySQL Installation and Configuration

Install MySQL Server:
```
apt install mysql-server
```

### Check MySQL Service Status
```
systemctl status mysql.service
```
```
systemctl enable mysql.service
```
```
netstat -nltup | grep mysql
```

### Start and Restart MySQL Service
```
systemctl start mysql.service

systemctl restart mysql.service

netstat -nltup | grep mysql
```

### Secure MySQL Installation
```
mysql_secure_installation
```

Follow the prompts to set a root password, remove anonymous users, disallow root login remotely, remove test database, and reload privileges.

Example new password:
```
p@ssW0rd@1234
```

### Login with MySQL Root Password
```
mysql -u root -p
```

Enter your password when prompted.

### Change Root Password (If Needed)
```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Armour@123';
```
```
SHOW DATABASES;
```
```
EXIT;
```

## phpMyAdmin Installation and Configuration

### Install phpMyAdmin from Debian Repository
```
apt install phpmyadmin
```

This will automatically:
- Install phpMyAdmin and dependencies
- Ask you to choose web server (select apache2)
- Ask if you want to configure database with dbconfig-common (usually select Yes)
- Ask for a password for phpMyAdmin database

Alternatively, you can download from the official website:

### Download and Extract phpMyAdmin Manually
```
wget https://files.phpmyadmin.net/phpMyAdmin/5.2.1/phpMyAdmin-5.2.1-all-languages.zip
```
```
apt install unzip
unzip phpMyAdmin-5.2.1-all-languages.zip
```
```
mkdir /var/www/html/phpmyadmin
```
```
cp -vr phpMyAdmin-5.2.1-all-languages/* /var/www/html/phpmyadmin/
```
```
cd /var/www/html/
```
```
cp /var/www/html/phpmyadmin/config.sample.inc.php /var/www/html/phpmyadmin/config.inc.php
```
```
vim /var/www/html/phpmyadmin/config.inc.php
```

Generate a random secret passphrase:
```
apt install pwgen
pwgen 32 -1
```

Add the generated password in config.inc.php:
```
####line 16 
$cfg['blowfish_secret'] = 'eephoo8ey8EuQu6Jiewee1ietaew6Eit'; /* YOU MUST FILL IN THIS FOR COOKIE AUTH! */
```

Set proper ownership:
```
chown -Rv www-data:www-data /var/www/html/phpmyadmin
```

Restart Apache:
```
systemctl restart apache2.service
```

Access phpMyAdmin via:
http://your-server-ip/phpmyadmin
