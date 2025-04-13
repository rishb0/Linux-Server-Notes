# WordPress Installation and Configuration

WordPress is a popular content management system (CMS) used to create and manage websites.

### Step 1: Download and Extract WordPress

Download the latest version of WordPress from the official website:
```
wget https://wordpress.org/latest.zip
```

Extract the downloaded package:
```
unzip latest.zip
```

Move WordPress files to the Apache web directory:
```
cp -vr wordpress/ /var/www/html/
```

Navigate to the web root directory:
```
cd /var/www/html/
```

Set the correct ownership to allow Apache to manage WordPress files:
```
chown -Rv apache:apache wordpress/
```

Navigate to the WordPress directory:
```
cd wordpress/
```

Set proper permissions for essential WordPress directories:
```
chmod -Rv 0755 wp-includes/ wp-admin/js/ wp-content/themes/ wp-content/plugins/
``` 
### Step 2: Configure Apache for WordPress

Open the Apache configuration file for editing:
```
vim /etc/httpd/conf/httpd.conf
```

```
<VirtualHost 192.168.1.31:443>
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/localhost.crt
    SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
    DocumentRoot /var/www/html/wordpress/
    DirectoryIndex index.php
</VirtualHost>
```

Restart Apache to apply changes:
```
systemctl restart httpd.service
```

Restart firewall services if necessary:
```
systemctl restart iptables.service
```

Restart DNS service if applicable:
```
systemctl restart named.service
```

Test domain resolution to ensure WordPress is accessible:
```
dig armour.local
```
# Apache VirtualHost Configuration

A VirtualHost configuration allows Apache to serve multiple websites from the same server.

### Step 1: Configure VirtualHost for SSL

To enable HTTPS support for your website, add the following configuration in the Apache Virtual Host file:

```apache
<VirtualHost 192.168.1.31:443>
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/localhost.crt
    SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
    ServerName armour.local
    DocumentRoot /var/www/html/wordpress/
    DirectoryIndex index.php
    ServerAlias www.armour.local
</VirtualHost>
```

### Step 2: Configure VirtualHost for HTTP

If you want your site to be accessible over HTTP (non-secure), use this configuration:

```apache
<VirtualHost 192.168.1.31:80>
    ServerName armour.local
    DocumentRoot /var/www/html/wordpress/
    DirectoryIndex index.php
    ServerAlias www.armour.local
</VirtualHost>
```
