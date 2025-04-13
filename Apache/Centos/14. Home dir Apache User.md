# Enable Apache User Home Directories

Apache allows individual users to host web content from their home directories using the `mod_userdir` module. This setup is useful for development, personal websites, or segregated hosting.

### 1. Enable UserDir in Apache Configuration

Edit the `userdir.conf` File:
```
vim /etc/httpd/conf.d/userdir.conf
```

Sample Configuration:
```apache
<IfModule mod_userdir.c>
    UserDir enabled
    UserDir public_html
</IfModule>

<Directory "/home/*/public_html">
    AllowOverride FileInfo AuthConfig Limit Indexes
    Options MultiViews Indexes SymLinksIfOwnerMatch IncludesNoExec
    Require method GET POST OPTIONS
</Directory>
```

This configuration:
- Enables `public_html` inside each user's home directory.
- Allows basic options like indexing, symbolic links, and overrides.
- Limits requests to safe HTTP methods. 

### 2. Configure Home Directory for User: armour

Create Public Web Directory:
```
mkdir /home/armour/public_html
```

Set Correct Permissions:
```
chmod 711 /home/armour/
```
```
chown -R armour:armour /home/armour/public_html/
```
```
chmod -R 755 /home/armour/public_html/
```

Note: `711` on the home folder ensures the Apache process can enter it, while `755` on the web folder allows content access.
### 3. Create VirtualHost for armour

Edit the VirtualHost File:
```
vim /etc/httpd/conf.d/armour.conf
```

Example VirtualHost Configuration:
```apache
<VirtualHost 192.168.1.37:80>
    DocumentRoot /home/armour/public_html
    DirectoryIndex index.html
</VirtualHost>
```

This binds the site to the server IP `192.168.1.37` and serves content from `armour`'s public directory.
### 4. Configure Home Directory for Another User: infosec

Create Public Web Directory:
```
mkdir /home/infosec/public_html
```

Set Correct Permissions:
```
chmod 711 /home/infosec/
```
```
chown -R infosec:infosec /home/infosec/public_html/
```
```
chmod -R 755 /home/infosec/public_html/
```
### 5. Create VirtualHost for infosec

Edit the VirtualHost File:
```
vim /etc/httpd/conf.d/infosec.conf
```

Example HTTP and HTTPS VirtualHosts:

```apache
<VirtualHost 192.168.1.37:80>
    ServerName infosec.com
    DocumentRoot /home/infosec/public_html/
    DirectoryIndex index.html
    ServerAlias www.infosec.com
    <Directory /home/infosec/public_html/ >
        Options -Indexes
    </Directory>
</VirtualHost>
```

```
<VirtualHost 192.168.1.37:443>
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/ca.crt
    SSLCertificateKeyFile /etc/pki/tls/private/ca.key
    ServerName armour.com
    DocumentRoot /home/infosec/public_html/
    DirectoryIndex index.html
    ServerAlias www.armour.com
    <Directory /home/infosec/public_html/ >
        Options -Indexes
    </Directory>
</VirtualHost>
```
- The HTTP VirtualHost serves `infosec.com`
- The HTTPS VirtualHost uses SSL to serve `armour.com` from the same directory (can be adjusted per site).

### 6. Restart Apache to Apply Changes
```
systemctl restart httpd
```

### Notes
- Make sure SELinux (if enabled) is configured to allow home directory access by Apache.
- Ensure `mod_userdir` is loaded in Apache (enabled by default on most systems)
- Consider firewall rules or DNS mappings to test domain-based VirtualHosts on a local network.

# **7. SELinux Configuration (If Enabled)**

If SELinux is enforcing, Apache **won’t be able to access `/home/username/public_html`** unless you configure proper contexts.

### Solution:

`setsebool -P httpd_enable_homedirs 1 chcon -R -t httpd_sys_content_t /home/username/public_html`

Or permanently via `semanage fcontext` if installed.