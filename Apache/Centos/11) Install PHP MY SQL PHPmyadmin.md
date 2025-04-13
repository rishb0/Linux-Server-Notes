# PHP-and-Mysql-Installation-and-Configuration
## PHP Installation and Configuration

### Install Required Repositories

```
yum install epel-release yum-utils
```
```
yum repolist all
```

Visit the official Remi repository for the latest repository package:

[Remi Repository](http://rpms.remirepo.net/enterprise/remi-release-9.rpm)

```
yum install http://rpms.remirepo.net/enterprise/remi-release-9.rpm
```
```
yum repolist all
```
```
rpm -qc remi-release
```
```
rpm -ql remi-release
```
```
vim /etc/yum.repos.d/remi.repo
```
```
yum repolist all
```

### Search for Available PHP Versions
```
yum search php
```
```
yum info php.x86_64
```
```
yum info php81.x86_64
```
```
yum info php84.x86_64
```

### Install PHP and Required Extensions
```
yum install php php-common.x86_64 php-cli.x86_64 php-opcache.x86_64 php-gd.x86_64 php-curl php-mysqlnd.x86_64 php-xml.x86_64 php-mbstring.x86_64 php-pear php-mbstring php-pecl-http php-session
```

### Verify PHP Installation
```
php -v
```

### Restart Apache
```
systemctl restart httpd.service
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

Download and Install MySQL Repository:
```
wget https://dev.mysql.com/get/mysql84-community-release-el9-1.noarch.rpm
```
```
dnf install ./mysql84-community-release-el9-1.noarch.rpm
```
```
dnf repolist all
```
### Enable and Disable MySQL Versions
```
yum-config-manager --enable mysql80.community
```
```
yum-config-manager --enable mysql57.community
```
```
yum-config-manager --disable mysql57.community
```

### Search for MySQL Packages
```
yum search mysql
```

### Install MySQL Server and Development Packages
```
yum install mysql-community-server mysql-community-devel
```

### Check MySQL Service Status
```
systemctl status mysqld.service
```
```
systemctl enable mysqld.service
```
```
netstat -nltup | grep mysql
```

### Start and Restart MySQL Service
```
systemctl start mysqld.service

systemctl restart mysqld.service

netstat -nltup | grep mysql
```

### Retrieve MySQL Root Password
```
ls -lh /var/log/mysqld.log
```
```
grep 'temporary password' /var/log/mysqld.log
```

Example Output:
```
A temporary password is generated for root@localhost: s;r=oAtZ5UuR
```

### Secure MySQL Installation
```
mysql -u root -p
```

Enter the temporary password when prompted.

```
mysql_secure_installation
```

Example new password:
```
p@ssW0rd@1234
```

### Login with New MySQL Root Password
```
mysql -u root -p  p@ssW0rd@1234
```

### Change Root Password (If Needed)
```
ALTER USER root@localhost IDENTIFIED WITH mysql_native_password BY 'Armour@123';
```
```
SHOW DATABASES;
```
```
EXIT;
```

## phpMyAdmin Installation and Configuration

Visit the official phpMyAdmin website:
[phpMyAdmin](https://www.phpmyadmin.net)
### Download and Extract phpMyAdmin
```
wget https://files.phpmyadmin.net/phpMyAdmin/5.1.0/phpMyAdmin-5.1.0-all-languages.zip
```
```
unzip phpMyAdmin-5.1.0-all-languages.zip
```
```
mkdir /var/www/html/phpmyadmin
```
```
cp -vr phpMyAdmin-5.1.0-all-languages/* /var/www/html/phpmyadmin/
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
pwgen 32 -1
```
- add the generated password in /var/www/html/phpmyadmin/config.inc.php 
```
####line 16 
-$cfg['blowfish_secret'] = 'eephoo8ey8EuQu6Jiewee1ietaew6Eit'; /* YOU MUST FILL IN THIS FOR COOKIE AUTH! */
```
Set proper ownership:
```
chown -Rv apache:apache /var/www/html/phpmyadmin
```
Restart Apache:
```
systemctl restart httpd.service
```
