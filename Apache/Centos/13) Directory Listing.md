# Directory Listing

### Change Directory and Create an Empty File
```
cd /wordpress/wp-content/uploads/2021/03
```
```
touch index.php
```

### Disable Directory Listing for All Directories in Apache

1. Edit the Apache configuration file:
```
vim /etc/httpd/conf/httpd.conf
```
```
<VirtualHost 192.168.1.35:443>
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/localhost.crt
    SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
    DocumentRoot /var/www/html/wordpress/
    DirectoryIndex index.php
    <Directory /var/www/html/wordpress >
        Options -Indexes
    </Directory>
</VirtualHost>
```
2. Restart the Apache service:
```
systemctl restart httpd.service
```

3. Edit the specific site configuration file:
```
vim /etc/httpd/conf.d/wp-site.conf
```
```
<VirtualHost 192.168.1.35:443>
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/localhost.crt
    SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
    ServerName armour.local
    DocumentRoot /var/www/html/wordpress/
    DirectoryIndex index.php
    ServerAlias www.armour.local
    <Directory /var/www/html/wordpress >
        Options -Indexes
    </Directory>
</VirtualHost>
```
4. Restart Apache again:
```
systemctl restart httpd.service
```
### Enable Directory Listing for a Selected Directory

1. Create a new directory:
```
mkdir /var/www/html/wordpress/backup
```

2. Change ownership:
```
chown -R apache:apache /var/www/html/wordpress/backup
```

3. Edit the Apache configuration:
```
vim /etc/httpd/conf/httpd.conf
```
```apache
<VirtualHost 192.168.1.35:443>
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/localhost.crt
    SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
    DocumentRoot /var/www/html/wordpress/
    DirectoryIndex index.php
    <Directory /var/www/html/wordpress/backup >
        Options Indexes
    </Directory>
    <Directory /var/www/html/wordpress >
        Options -Indexes
    </Directory>
</VirtualHost>
```
4. Restart Apache:
```
systemctl restart httpd.service
```

5. View the specific site configuration:
```
cat /etc/httpd/conf.d/wp-site.conf
```

### Allow Selected IP Addresses

1. Edit Apache configuration:
```
vim /etc/httpd/conf/httpd.conf
```
- for apache 2.4 or older versions
```apache
<VirtualHost 192.168.1.35:443>
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/localhost.crt
    SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
    
    DocumentRoot /var/www/html/wordpress/
    DirectoryIndex index.php
    
    <Directory /var/www/html/wordpress/backup >
        Options Indexes
        Order allow,deny
        Allow from 192.168.1.7 192.168.1.51
    </Directory>
    
    <Directory /var/www/html/wordpress >
        Options -Indexes
    </Directory>
</VirtualHost>
```
- for apache new versions
```
<VirtualHost 192.168.1.35:443>
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/localhost.crt
    SSLCertificateKeyFile /etc/pki/tls/private/localhost.key

    DocumentRoot /var/www/html/wordpress/
    DirectoryIndex index.php

    <Directory /var/www/html/wordpress/backup>
        Options Indexes
        Require ip 192.168.1.7 192.168.1.51
    </Directory>

    <Directory /var/www/html/wordpress>
        Options -Indexes
        Require all granted  #add this if not allowd by defult configurations
    </Directory>
</VirtualHost>
```
2. Restart Apache:
```
systemctl restart httpd.service
```

### Deny Selected IP Addresses

1. Edit Apache configuration:
```
vim /etc/httpd/conf/httpd.conf
```
- for older version of apache below 2.4
```apache
<VirtualHost 192.168.1.35:443>
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/localhost.crt
    SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
    DocumentRoot /var/www/html/wordpress/
    DirectoryIndex index.php
    <Directory /var/www/html/wordpress/backup >
        Options Indexes
        Order allow,deny
        Allow from all
        Deny from 192.168.1.51
    </Directory>
    <Directory /var/www/html/wordpress >
        Options -Indexes
    </Directory>
</VirtualHost>
```
- for new versions above 2.4
```
<VirtualHost 192.168.1.101:443>
    DocumentRoot /var/www/html/wordpress/
    DirectoryIndex index.php
    SSLEngine on
    SSLCertificateFile /opt/ssl/rscert.pem
    SSLCertificateKeyFile /opt/ssl/rskey.pem

    <Directory /var/www/html/wordpress/backup>
        Options Indexes
        <RequireAll>
          Require all granted
          Require not ip 192.168.1.8
        </RequireAll>
    </Directory>

	<Directory /var/www/html/wordpress/>
        Options -Indexes
    </Directory>

</VirtualHost>
```
2. Restart Apache:
```
systemctl restart httpd.service
```

### Secure Directory Hosting with User Authentication

#### Using .htaccess

1. Edit the .htaccess file:
```
vim /var/www/html/wordpress/backup/.htaccess
```

Add the following content:
```apache
AuthName "Armour Infosec"
AuthType basic
AuthUserFile /etc/httpd/htpasswd
Require valid-user
```

2. Create a password file and add users:
```
htpasswd -c /etc/httpd/htpasswd user1
```
- The `-c` will create the file. So use it when running this command for the first time.

```
htpasswd /etc/httpd/htpasswd user2
```

View the password file contents:
```
cat /etc/httpd/htpasswd
```

Output shows encrypted passwords for users:
```
user1:$apr1$e51zqEpL$hjMvlTCuyGIKcRS2ISdt1
user2:$apr1$UIIwlefV$G7c78m4F2OH6gQeTE7bF/
```

3. Edit the Apache configuration:
```
vim /etc/httpd/conf/httpd.conf
```

```apache
<VirtualHost 192.168.1.35:443>
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/localhost.crt
    SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
    DocumentRoot /var/www/html/wordpress/
    DirectoryIndex index.php
    <Directory /var/www/html/wordpress/backup >
        Options Indexes
        AllowOverride Authconfig
    </Directory>
    <Directory /var/www/html/wordpress >
        Options -Indexes
    </Directory>
</VirtualHost>
```
If using this then delete the `.htaccess` file 
4. Restart Apache:
```
systemctl restart httpd.service
```

### User Authentication Without .htaccess

1. Edit the Apache configuration:
```
vim /etc/httpd/conf/httpd.conf
```

```apache
<VirtualHost 192.168.1.35:443>
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/localhost.crt
    SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
    DocumentRoot /var/www/html/wordpress/
    DirectoryIndex index.php
    <Directory /var/www/html/wordpress/backup >
        AuthName "Private"
        AuthType Basic
        AuthBasicProvider file
        AuthUserFile /etc/httpd/htpasswd
        Require valid-user
        Options +Indexes
    </Directory>
    <Directory /var/www/html/wordpress >
        Options -Indexes
    </Directory>
</VirtualHost>
```

```
<VirtualHost 192.168.1.101:443>
    DocumentRoot /var/www/html/wordpress/
    DirectoryIndex index.php
    SSLEngine on
    SSLCertificateFile /opt/ssl/rscert.pem
    SSLCertificateKeyFile /opt/ssl/rskey.pem

    <Directory /var/www/html/wordpress/>
        Options -Indexes
    </Directory>

    <Directory /var/www/html/wordpress/backup>
        Options Indexes
     <RequireAll>
          Require all granted
          Require not ip 192.168.1.8
     Require valid-user
     </RequireAll>
     AuthName "Rishabh backup auth"
     AuthType basic
     AuthBasicProvider file
     AuthUserFile /etc/httpd/htpasswd
     </Directory>

</VirtualHost>
```
2. Restart Apache:
```
systemctl restart httpd.service
```