# Directory Listing

### Change Directory and Create an Empty File
```
cd /var/www/html/wordpress/wp-content/uploads/2021/03
```
```
touch index.php
```

### Disable Directory Listing for All Directories in Apache

1. Create or edit a site configuration file:
```
vim /etc/apache2/sites-available/wordpress-ssl.conf
```
```
<VirtualHost 192.168.1.35:443>
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/localhost.crt
    SSLCertificateKeyFile /etc/ssl/private/localhost.key
    DocumentRoot /var/www/html/wordpress/
    DirectoryIndex index.php
    <Directory /var/www/html/wordpress >
        Options -Indexes
    </Directory>
</VirtualHost>
```
2. Enable the site and restart the Apache service:
```
a2ensite wordpress-ssl.conf
systemctl restart apache2.service
```

3. Alternative: Edit a specific site configuration file:
```
vim /etc/apache2/sites-available/wp-site.conf
```
```
<VirtualHost 192.168.1.35:443>
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/localhost.crt
    SSLCertificateKeyFile /etc/ssl/private/localhost.key
    ServerName armour.local
    DocumentRoot /var/www/html/wordpress/
    DirectoryIndex index.php
    ServerAlias www.armour.local
    <Directory /var/www/html/wordpress >
        Options -Indexes
    </Directory>
</VirtualHost>
```
4. Enable the site and restart Apache again:
```
a2ensite wp-site.conf
systemctl restart apache2.service
```

### Enable Directory Listing for a Selected Directory

1. Create a new directory:
```
mkdir /var/www/html/wordpress/backup
```

2. Change ownership:
```
chown -R www-data:www-data /var/www/html/wordpress/backup
```

3. Edit the Apache configuration:
```
vim /etc/apache2/sites-available/wordpress-ssl.conf
```
```apache
<VirtualHost 192.168.1.35:443>
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/localhost.crt
    SSLCertificateKeyFile /etc/ssl/private/localhost.key
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
systemctl restart apache2.service
```

5. View the specific site configuration:
```
cat /etc/apache2/sites-available/wp-site.conf
```

### Allow Selected IP Addresses

1. Edit Apache configuration:
```
vim /etc/apache2/sites-available/wordpress-ssl.conf
```
- For Apache 2.2 or older versions:
```apache
<VirtualHost 192.168.1.35:443>
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/localhost.crt
    SSLCertificateKeyFile /etc/ssl/private/localhost.key
    
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
- For Apache 2.4 (default in Debian):
```
<VirtualHost 192.168.1.35:443>
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/localhost.crt
    SSLCertificateKeyFile /etc/ssl/private/localhost.key

    DocumentRoot /var/www/html/wordpress/
    DirectoryIndex index.php

    <Directory /var/www/html/wordpress/backup>
        Options Indexes
        Require ip 192.168.1.7 192.168.1.51
    </Directory>

    <Directory /var/www/html/wordpress>
        Options -Indexes
        Require all granted  #add this if not allowed by default configurations
    </Directory>
</VirtualHost>
```
2. Restart Apache:
```
systemctl restart apache2.service
```

### Deny Selected IP Addresses

1. Edit Apache configuration:
```
vim /etc/apache2/sites-available/wordpress-ssl.conf
```
- For Apache 2.2 or older versions:
```apache
<VirtualHost 192.168.1.35:443>
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/localhost.crt
    SSLCertificateKeyFile /etc/ssl/private/localhost.key
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
- For Apache 2.4 (default in Debian):
```
<VirtualHost 192.168.1.101:443>
    DocumentRoot /var/www/html/wordpress/
    DirectoryIndex index.php
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/rscert.pem
    SSLCertificateKeyFile /etc/ssl/private/rskey.pem

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
systemctl restart apache2.service
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
AuthUserFile /etc/apache2/htpasswd
Require valid-user
```

2. Create a password file and add users:
```
htpasswd -c /etc/apache2/htpasswd user1
```
- The `-c` will create the file. So use it when running this command for the first time.

```
htpasswd /etc/apache2/htpasswd user2
```

View the password file contents:
```
cat /etc/apache2/htpasswd
```

Output shows encrypted passwords for users:
```
user1:$apr1$e51zqEpL$hjMvlTCuyGIKcRS2ISdt1
user2:$apr1$UIIwlefV$G7c78m4F2OH6gQeTE7bF/
```

3. Edit the Apache configuration:
```
vim /etc/apache2/sites-available/wordpress-ssl.conf
```

```apache
<VirtualHost 192.168.1.35:443>
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/localhost.crt
    SSLCertificateKeyFile /etc/ssl/private/localhost.key
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
If using the above method, you need the .htaccess file. If you prefer to configure authentication directly in the Apache configuration (see below), delete the .htaccess file.

4. Make sure the auth_basic module is enabled:
```
a2enmod auth_basic
```

5. Restart Apache:
```
systemctl restart apache2.service
```

### User Authentication Without .htaccess

1. Edit the Apache configuration:
```
vim /etc/apache2/sites-available/wordpress-ssl.conf
```

```apache
<VirtualHost 192.168.1.35:443>
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/localhost.crt
    SSLCertificateKeyFile /etc/ssl/private/localhost.key
    DocumentRoot /var/www/html/wordpress/
    DirectoryIndex index.php
    <Directory /var/www/html/wordpress/backup >
        AuthName "Private"
        AuthType Basic
        AuthBasicProvider file
        AuthUserFile /etc/apache2/htpasswd
        Require valid-user
        Options +Indexes
    </Directory>
    <Directory /var/www/html/wordpress >
        Options -Indexes
    </Directory>
</VirtualHost>
```

Another example with combined access controls:
```
<VirtualHost 192.168.1.101:443>
    DocumentRoot /var/www/html/wordpress/
    DirectoryIndex index.php
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/rscert.pem
    SSLCertificateKeyFile /etc/ssl/private/rskey.pem

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
     AuthUserFile /etc/apache2/htpasswd
     </Directory>

</VirtualHost>
```
2. Make sure the auth_basic module is enabled:
```
a2enmod auth_basic
```

3. Restart Apache:
```
systemctl restart apache2.service
```
