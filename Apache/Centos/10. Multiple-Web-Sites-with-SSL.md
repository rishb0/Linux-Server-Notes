# Multiple-Web-Sites-with-SSL

## 1. Apache SSL VirtualHost Configuration

### 1: Editing the Virtual Host Configurations

1. Edit the configuration file (`site2.conf` or any other configuration in `/etc/httpd/conf.d/`) to include the virtual hosts for SSL on ports 8080 and 8443.

```
vim /etc/httpd/conf.d/site2.conf
```

2. Add SSL Virtual Hosts for ports 8080 and 8443. Here is the configuration you provided:

Virtual Host for Port 8080:
```
Listen 8080

<VirtualHost 192.168.1.34:8080>
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/ca.crt
    SSLCertificateKeyFile /etc/pki/tls/private/ca.key
    ServerName armour.com
    DocumentRoot /var/www/html/site2/
    DirectoryIndex index.html
    ServerAlias www.ai.local
</VirtualHost>
```

Virtual Host for Port 8443:
```
Listen 8443

<VirtualHost 192.168.1.34:8443>
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/ca.crt
    SSLCertificateKeyFile /etc/pki/tls/private/ca.key
    ServerName armour.local
    DocumentRoot /var/www/html/site1/
    DirectoryIndex index.html
    ServerAlias www.armour.local
</VirtualHost>
```

3. Save and close the configuration file after making the changes.
## 2. Firewall Configuration

Next, you'll want to make sure the firewall is configured to allow traffic on these custom SSL ports (8080 and 8443).

1. Allow traffic on port 8080 (SSL):
```
firewall-cmd --permanent --add-port=8080/tcp
```

2. Allow traffic on port 8443 (SSL):
```
firewall-cmd --permanent --add-port=8443/tcp
```

3. Reload the firewall to apply the changes:
```
firewall-cmd --reload
```

## 3. Restart Apache

After editing the configuration and updating the firewall, restart Apache to apply the changes.

```
systemctl restart httpd
```

## 4. Verify Ports Are Open

Finally, you can check if Apache is properly listening on the new SSL ports (8080 and 8443).

```
netstat -nltup | grep httpd
```

Look for output showing Apache listening on `8080` and `8443` (it should look something like `tcp6 0 0 :::8080 :::* LISTEN`).

### Final Checks

* **Test Access**: You should now be able to access the sites via:
    * `https://ai.local:8080` (for port 8080)
    * `https://armour.local:8443` (for port 8443)
    Check if SSL is active by ensuring the padlock icon is visible in the browser for `https`.

* **Verify Certificates**: Make sure your certificates are correctly installed and valid.
