# Binding-with-Type(SSL-TLS)
## 1. Install and Configure SSL/TLS in Apache

### Step 1: Install mod_ssl

mod_ssl is an Apache module that provides SSL and TLS support.

    yum install mod_ssl

### Step 2: Verify mod_ssl Installation

Check if the module is installed:

    rpm -qi mod_ssl

	rpm -ql mod_ssl
    
    rpm -qd mod_ssl

### Step 3: Configure SSL in Apache

Edit the SSL configuration file:

    vim /etc/httpd/conf.d/ssl.conf

Make sure that SSL is enabled by setting appropriate certificate files:

    SSLCertificateFile /etc/pki/tls/certs/localhost.crt
    
	SSLCertificateKeyFile /etc/pki/tls/private/localhost.key

### Step 4: Restart Apache

Restart Apache to apply SSL configuration:

    systemctl restart httpd.service

### Step 5: Verify Apache Listening on SSL Port

Check if Apache is listening on port 443 (SSL):

    netstat -nltup | grep httpd

### Step 6: Update Firewall for SSL

Edit the firewall rules to allow SSL traffic (port 443):

    firewall-cmd --permanent --add-port=443/tcp

	firewall-cmd --permanent --add-port=8080/tcp
    
    firewall-cmd --reload

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

    cp -v /opt/ssl/cert.pem /etc/pki/tls/certs/cert.pem
    
    cp -v /opt/ssl/key.pem /etc/pki/tls/private/key.pem

These commands verify the creation of the SSL certificate and private key files, then copy them to the standard system locations where Apache expects to find SSL certificates and keys.

## 3. Update Apache Configuration with SSL Certificates

Edit the Apache configuration to point to the newly created certificates:

    vim /etc/httpd/conf/httpd.conf

Add these lines to enable SSL:

    # SSL Cipher Suite:
    # List the ciphers that the client is permitted to negotiate.
    # See the mod_ssl documentation for a complete list.
    # The OpenSSL system profile is configured by default. See
    # update-crypto-policies(8) for more details.
    SSLCipherSuite PROFILE=SYSTEM
    SSLProxyCipherSuite PROFILE=SYSTEM

    # Point SSLCertificateFile at a PEM encoded certificate. If
    # the certificate is encrypted, then you will be prompted for a
    # pass phrase. Note that restarting httpd will prompt again. Keep
    # in mind that if you have both an RSA and a DSA certificate you
    # can configure both in parallel (to also allow the use of DSA
    # ciphers, etc.)
    # Some ECC cipher suites (http://www.ietf.org/rfc/rfc4492.txt)
    # require an ECC certificate which can also be configured in
    # parallel.
    #SSLCertificateFile /etc/pki/tls/certs/localhost.crt
    SSLCertificateFile /etc/pki/tls/certs/cert.pem

    # Server Private Key:
    # If the key is not combined with the certificate, use this
    # directive to point at the key file. Keep in mind that if
    # you've both a RSA and a DSA private key you can configure
    # both in parallel (to also allow the use of DSA ciphers, etc.)
    # ECC keys, when in use, can also be configured in parallel
    #SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
    SSLCertificateKeyFile /etc/pki/tls/private/key.pem

Restart Apache again:

    systemctl restart httpd.service

## 4. Verify SSL Binding in Apache

Check if Apache is listening on both HTTP (port 80) and HTTPS (port 443):

    netstat -nltup | grep httpd

## 5. Update Firewall for SSL/TLS Ports

Ensure that port 443 is open in your firewall configuration:

    firewall-cmd --permanent --add-port=443/tcp

    firewall-cmd --permanent --add-port=8080/tcp

    firewall-cmd --reload

# 6. Configure Apache Virtual Hosts with SSL/TLS

You can now create Virtual Hosts with SSL/TLS enabled. Below are example configurations for both HTTP and HTTPS.

### Example of Apache Virtual Hosts for HTTP and HTTPS

```
<VirtualHost 192.168.1.31:80>
    DocumentRoot /var/www/html/site1/
    DirectoryIndex index.html
</VirtualHost>

<VirtualHost 192.168.1.31:443>
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/cert.pem
    SSLCertificateKeyFile /etc/pki/tls/private/key.pem
    DocumentRoot /var/www/html/site1/
    DirectoryIndex index.html
</VirtualHost>

<VirtualHost 192.168.1.31:80>
    ServerName armour.local
    DocumentRoot /var/www/html/site2/
    DirectoryIndex index.html
    ServerAlias www.armour.local
</VirtualHost>

<VirtualHost 192.168.1.31:443>
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/cert.pem
    SSLCertificateKeyFile /etc/pki/tls/private/key.pem
    ServerName armour.local
    DocumentRoot /var/www/html/site2/
    DirectoryIndex index.html
    ServerAlias www.armour.local
</VirtualHost>
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
cp -v /opt/ssl/ca.crt /etc/pki/tls/certs/ca.crt
cp -v /opt/ssl/ca.key /etc/pki/tls/private/ca.key
```
### Step 5: Configure Apache Virtual Hosts with SSL/TLS

You can now create Virtual Hosts with SSL/TLS enabled. Below are example configurations for both HTTP and HTTPS.

```
vim /etc/httpd/conf/httpd.conf

Example of Apache Virtual Hosts for HTTP and HTTPS**

<VirtualHost 192.168.1.34:443>
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/ca.crt
    SSLCertificateKeyFile /etc/pki/tls/private/ca.key
    DocumentRoot /var/www/html/site1/
    DirectoryIndex index.html
</VirtualHost>
```

# 8. Final Steps

After configuring your Virtual Hosts and SSL certificates, restart Apache and check the ports once again:

```
systemctl restart httpd.service
netstat -nltup | grep httpd
```

Test your websites using both HTTP and HTTPS URLs:

| Domain | Port | Expected Output |
|--------|------|----------------|
| http://armour.local | 80 | site2 homepage |
| https://armour.local | 443 | site2 (SSL-enabled) homepage |

# 🏆 SSL/TLS Binding is Now Complete! 😎

You've successfully set up SSL and TLS in Apache, making your site secure with HTTPS!

