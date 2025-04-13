# Multiple-Web-Sites-with-SSL

## 1. Apache SSL VirtualHost Configuration

### 1: Editing the Virtual Host Configurations

1. First, add the listening ports to Apache's ports configuration file:

```
vim /etc/apache2/ports.conf
```

Add these lines:
```
Listen 8080
Listen 8443
```

2. Create configuration files for your SSL sites in the sites-available directory:

```
vim /etc/apache2/sites-available/armour-8080-ssl.conf
```

Virtual Host for Port 8080:
```
<VirtualHost 192.168.1.34:8080>
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/ca.crt
    SSLCertificateKeyFile /etc/ssl/private/ca.key
    ServerName armour.com
    DocumentRoot /var/www/html/site2/
    DirectoryIndex index.html
    ServerAlias www.ai.local
</VirtualHost>
```

3. Create the second virtual host configuration:

```
vim /etc/apache2/sites-available/armour-8443-ssl.conf
```

Virtual Host for Port 8443:
```
<VirtualHost 192.168.1.34:8443>
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/ca.crt
    SSLCertificateKeyFile /etc/ssl/private/ca.key
    ServerName armour.local
    DocumentRoot /var/www/html/site1/
    DirectoryIndex index.html
    ServerAlias www.armour.local
</VirtualHost>
```

4. Enable the SSL module if not already enabled:
```
a2enmod ssl
```

5. Enable the sites:
```
a2ensite armour-8080-ssl.conf
a2ensite armour-8443-ssl.conf
```

## 2. Firewall Configuration

Next, you'll want to make sure the firewall is configured to allow traffic on these custom SSL ports (8080 and 8443).

1. Allow traffic on port 8080 (SSL):
```
ufw allow 8080/tcp
```

2. Allow traffic on port 8443 (SSL):
```
ufw allow 8443/tcp
```

3. Check the firewall status:
```
ufw status
```

## 3. Restart Apache

After editing the configuration and updating the firewall, restart Apache to apply the changes.

```
systemctl restart apache2
```

## 4. Verify Ports Are Open

Finally, you can check if Apache is properly listening on the new SSL ports (8080 and 8443).

```
netstat -nltup | grep apache2
```

Look for output showing Apache listening on `8080` and `8443` (it should look something like `tcp6 0 0 :::8080 :::* LISTEN`).

### Final Checks

* **Test Access**: You should now be able to access the sites via:
    * `https://ai.local:8080` (for port 8080)
    * `https://armour.local:8443` (for port 8443)
    Check if SSL is active by ensuring the padlock icon is visible in the browser for `https`.

* **Verify Certificates**: Make sure your certificates are correctly installed and valid.

* **Check Apache Error Logs**: If you encounter any issues:
```
tail -f /var/log/apache2/error.log
```
## 5. Redirecting HTTP to HTTPS

For security reasons, you might want to redirect all HTTP traffic to HTTPS. This ensures users always connect securely to your websites.

### Create HTTP Virtual Hosts with Redirects

1. For the first site (Port 8080):

```
vim /etc/apache2/sites-available/armour-8080-redirect.conf
```

Add the following:
```
<VirtualHost 192.168.1.34:80>
    ServerName armour.com
    ServerAlias www.ai.local
    Redirect permanent / https://armour.com:8080/
</VirtualHost>
```

2. For the second site (Port 8443):

```
vim /etc/apache2/sites-available/armour-8443-redirect.conf
```

Add the following:
```
<VirtualHost 192.168.1.34:80>
    ServerName armour.local
    ServerAlias www.armour.local
    Redirect permanent / https://armour.local:8443/
</VirtualHost>
```

3. Enable these sites:

```
a2ensite armour-8080-redirect.conf
a2ensite armour-8443-redirect.conf
systemctl reload apache2
```

## 6. Strengthening SSL Security

Apache's default SSL configuration may not provide the strongest security. You can enhance it by:

### 1. Creating a Strong SSL Configuration

```
vim /etc/apache2/conf-available/ssl-strong.conf
```

Add the following:
```
# Disable vulnerable SSL protocols
SSLProtocol all -SSLv2 -SSLv3 -TLSv1 -TLSv1.1

# Use strong ciphers
SSLCipherSuite HIGH:!aNULL:!MD5:!3DES
SSLHonorCipherOrder on

# Enable HSTS (HTTP Strict Transport Security)
Header always set Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"

# Disable compression for BREACH attack prevention
SSLCompression off

# Enable OCSP Stapling
SSLUseStapling on
SSLStaplingCache "shmcb:logs/stapling-cache(150000)"
```

### 2. Enable the Required Modules

```
a2enmod headers
a2enconf ssl-strong
systemctl reload apache2
```

## 7. Testing Your SSL Configuration

You should test your SSL configuration to ensure it's secure:

### Using OpenSSL to Test Connections

```
openssl s_client -connect armour.com:8080
openssl s_client -connect armour.local:8443
```

Look for details about the certificate and the connection parameters.

### Online SSL Testing Tools

You can use online tools to test your SSL configuration (if your server is publicly accessible):
- SSL Labs Server Test (https://www.ssllabs.com/ssltest/)
- ImmuniWeb SSL Security Test (https://www.immuniweb.com/ssl/)

## 8. Automatic Certificate Renewal

If you're using Let's Encrypt certificates (instead of self-signed), you'll want to set up automatic renewal:

### Install Certbot

```
apt install certbot python3-certbot-apache
```

### Set Up Automatic Renewal

Create a cron job to automatically renew certificates:

```
echo "0 3 * * * /usr/bin/certbot renew --quiet" | sudo tee -a /etc/crontab
```

## 9. Troubleshooting Common SSL Issues

If you encounter issues with your SSL configuration, check these common problems:

### Certificate File Permissions

```
chmod 644 /etc/ssl/certs/ca.crt
chmod 600 /etc/ssl/private/ca.key
```

### Check Apache Error Logs

```
tail -f /var/log/apache2/error.log
```

### Verify Certificate Chain

```
openssl verify -CAfile /etc/ssl/certs/ca.crt /etc/ssl/certs/ca.crt
```

### Check Certificate Information

```
openssl x509 -in /etc/ssl/certs/ca.crt -text -noout
```

## 10. Virtual Host Naming Best Practices

When running multiple SSL sites:

1. **Use Distinct ServerName Directives**: Ensure each virtual host has a unique `ServerName`.

2. **Properly Configure DNS**: Make sure all domain names resolve to your server's IP address.

3. **Use ServerAlias for Variants**: Include www and non-www versions of your domains.

4. **Consider Using Name-Based Virtual Hosting with SNI**: Modern browsers support Server Name Indication (SNI), allowing multiple SSL certificates on the same IP and port.

This completes your comprehensive guide to running multiple SSL-enabled websites on your Debian Apache server!
