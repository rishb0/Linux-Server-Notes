# Binding-with-Type(SSL-TLS)
## 1. Install and Configure SSL/TLS in Apache

### Step 1: Install mod_ssl

mod_ssl is an Apache module that provides SSL and TLS support.

    apt install ssl-cert

Apache in Debian already has the SSL module available, but not enabled by default. Enable it with:

    a2enmod ssl

### Step 2: Verify mod_ssl Installation

Check if the module is installed and enabled:

    dpkg -l | grep apache2
    
    ls -l /etc/apache2/mods-enabled/ssl*
    
    apache2ctl -M | grep ssl

### Step 3: Configure SSL in Apache

Edit the SSL configuration file:

    vim /etc/apache2/sites-available/default-ssl.conf

Make sure that SSL is enabled by setting appropriate certificate files:

    SSLCertificateFile /etc/ssl/certs/ssl-cert-snakeoil.pem
    
    SSLCertificateKeyFile /etc/ssl/private/ssl-cert-snakeoil.key

### Step 4: Enable the SSL site

Enable the default SSL site:

    a2ensite default-ssl.conf

### Step 5: Restart Apache

Restart Apache to apply SSL configuration:

    systemctl restart apache2.service

### Step 6: Verify Apache Listening on SSL Port

Check if Apache is listening on port 443 (SSL):

    netstat -nltup | grep apache2

### Step 7: Update Firewall for SSL

Edit the firewall rules to allow SSL traffic (port 443):

    ufw allow 443/tcp

    ufw allow 8080/tcp
    
    ufw reload

## 2. Create Self-Signed SSL Certificates

### Step 1: Create SSL Directory

Create a directory to store SSL certificates:

    mkdir /opt/ssl
    
    cd /opt/ssl

### Step 2: Generate SSL Certificate and Key

Generate a self-signed SSL certificate and key:

    openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365

### Step 3: Check the Generated Files

Verify the files:

    ls -lh
    
    file key.pem
    
    file cert.pem

### Step 4: Copy SSL Files to Appropriate Directories

Copy the certificate and key to the proper directories:

    cp -v /opt/ssl/cert.pem /etc/ssl/certs/cert.pem
    
    cp -v /opt/ssl/key.pem /etc/ssl/private/key.pem

These commands verify the creation of the SSL certificate and private key files, then copy them to the standard system locations where Apache expects to find SSL certificates and keys.

## 3. Update Apache Configuration with SSL Certificates

Create or edit a site configuration with SSL:

    vim /etc/apache2/sites-available/ssl-site.conf

Add these lines to enable SSL:

```
<VirtualHost *:443>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/cert.pem
    SSLCertificateKeyFile /etc/ssl/private/key.pem
    
    <Directory /var/www/html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Enable the site and restart Apache:

    a2ensite ssl-site.conf
    systemctl restart apache2.service

## 4. Verify SSL Binding in Apache

Check if Apache is listening on both HTTP (port 80) and HTTPS (port 443):

    netstat -nltup | grep apache2

## 5. Update Firewall for SSL/TLS Ports

Ensure that port 443 is open in your firewall configuration:

    ufw allow 443/tcp

    ufw allow 8080/tcp

    ufw status

# 6. Configure Apache Virtual Hosts with SSL/TLS

You can now create Virtual Hosts with SSL/TLS enabled. Below are example configurations for both HTTP and HTTPS.

### Example of Apache Virtual Hosts for HTTP and HTTPS

```
# /etc/apache2/sites-available/site1.conf
<VirtualHost 192.168.1.31:80>
    DocumentRoot /var/www/html/site1/
    DirectoryIndex index.html
</VirtualHost>

# /etc/apache2/sites-available/site1-ssl.conf
<VirtualHost 192.168.1.31:443>
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/cert.pem
    SSLCertificateKeyFile /etc/ssl/private/key.pem
    DocumentRoot /var/www/html/site1/
    DirectoryIndex index.html
</VirtualHost>

# /etc/apache2/sites-available/armour.conf
<VirtualHost 192.168.1.31:80>
    ServerName armour.local
    DocumentRoot /var/www/html/site2/
    DirectoryIndex index.html
    ServerAlias www.armour.local
</VirtualHost>

# /etc/apache2/sites-available/armour-ssl.conf
<VirtualHost 192.168.1.31:443>
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/cert.pem
    SSLCertificateKeyFile /etc/ssl/private/key.pem
    ServerName armour.local
    DocumentRoot /var/www/html/site2/
    DirectoryIndex index.html
    ServerAlias www.armour.local
</VirtualHost>
```

Enable the sites:

```
a2ensite site1.conf site1-ssl.conf armour.conf armour-ssl.conf
systemctl restart apache2.service
```

In the above configuration:

* The first block listens on HTTP (port 80) for `site1`.
* The second block enables SSL (port 443) for `site1`.
* Similarly, the third and fourth blocks handle HTTP and HTTPS for `armour.local`.

# 7. Advanced Configuration with Custom CA (Certificate Authority)

If you want to use a self-generated Certificate Authority (CA):

### Step 1: Generate the Private Key for the CA
```
openssl genrsa -out ca.key 2048
```

### Step 2: Generate the Certificate Signing Request (CSR)
```
openssl req -new -key ca.key -out ca.csr
```

### Step 3: Create the Self-Signed Certificate for the CA
```
openssl x509 -req -days 365 -in ca.csr -signkey ca.key -out ca.crt
```
### Step 4: Copy the CA Certificate and Key to the Appropriate Directories
```
cp -v /opt/ssl/ca.crt /etc/ssl/certs/ca.crt
cp -v /opt/ssl/ca.key /etc/ssl/private/ca.key
```
### Step 5: Configure Apache Virtual Hosts with SSL/TLS

You can now create Virtual Hosts with SSL/TLS enabled. Below are example configurations for both HTTP and HTTPS.

```
vim /etc/apache2/sites-available/site1-ca-ssl.conf

# Example of Apache Virtual Hosts with CA Cert
<VirtualHost 192.168.1.34:443>
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/ca.crt
    SSLCertificateKeyFile /etc/ssl/private/ca.key
    DocumentRoot /var/www/html/site1/
    DirectoryIndex index.html
</VirtualHost>
```

Enable the site:
```
a2ensite site1-ca-ssl.conf
```

# 8. Final Steps

After configuring your Virtual Hosts and SSL certificates, restart Apache and check the ports once again:

```
systemctl restart apache2.service
netstat -nltup | grep apache2
```

Test your websites using both HTTP and HTTPS URLs:

| Domain | Port | Expected Output |
|--------|------|----------------|
| http://armour.local | 80 | site2 homepage |
| https://armour.local | 443 | site2 (SSL-enabled) homepage |

# 🏆 SSL/TLS Binding is Now Complete! 😎

You've successfully set up SSL and TLS in Apache, making your site secure with HTTPS!