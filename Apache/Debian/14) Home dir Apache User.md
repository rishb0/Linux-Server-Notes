# Enable Apache User Home Directories

Apache allows individual users to host web content from their home directories using the `mod_userdir` module. This setup is useful for development, personal websites, or segregated hosting.

### 1. Enable UserDir in Apache Configuration

First, enable the userdir module:
```
a2enmod userdir
```

Edit the userdir configuration file:
```
vim /etc/apache2/mods-available/userdir.conf
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

Create a new VirtualHost file:
```
vim /etc/apache2/sites-available/armour.conf
```

Example VirtualHost Configuration:
```apache
<VirtualHost 192.168.1.37:80>
    DocumentRoot /home/armour/public_html
    DirectoryIndex index.html
</VirtualHost>
```

Enable the site:
```
a2ensite armour.conf
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

Create a new VirtualHost file:
```
vim /etc/apache2/sites-available/infosec.conf
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
    SSLCertificateFile /etc/ssl/certs/ca.crt
    SSLCertificateKeyFile /etc/ssl/private/ca.key
    ServerName armour.com
    DocumentRoot /home/infosec/public_html/
    DirectoryIndex index.html
    ServerAlias www.armour.com
    <Directory /home/infosec/public_html/ >
        Options -Indexes
    </Directory>
</VirtualHost>
```

Enable the site:
```
a2ensite infosec.conf
```

- The HTTP VirtualHost serves `infosec.com`
- The HTTPS VirtualHost uses SSL to serve `armour.com` from the same directory (can be adjusted per site).

### 6. Make sure SSL is enabled (for HTTPS configuration)
```
a2enmod ssl
```

### 7. Restart Apache to Apply Changes
```
systemctl restart apache2
```

### Notes
- AppArmor is typically used in Debian instead of SELinux and usually doesn't restrict Apache's access to user home directories by default.
- If you have issues with permissions, check if Apache can access the home directories and if proper file ownership is set.
- For serving sites from user directories, the URL format is typically: http://your-server/~username/
- Consider firewall rules or DNS mappings to test domain-based VirtualHosts on a local network.

### AppArmor Configuration (If Issues Occur)

If AppArmor is causing access issues (rare for this configuration), you might need to modify the Apache profile:

1. Check if AppArmor is active:
```
aa-status
```

2. If needed, edit the Apache profile:
```
vim /etc/apparmor.d/usr.sbin.apache2
```

3. Add the following line to allow access to user home directories:
```
/home/*/public_html/** r,
```

4. Reload AppArmor:
```
systemctl reload apparmor
```