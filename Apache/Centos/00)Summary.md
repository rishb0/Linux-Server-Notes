# Apache Quick Reference Guide for CentOS

A concise reference for common Apache configurations on CentOS systems.

## Installation & Basic Commands

```bash
# Install Apache
yum update
yum install httpd

# For CentOS 8/9
dnf update
dnf install httpd

# Start, stop, restart
systemctl start httpd
systemctl stop httpd
systemctl restart httpd
systemctl reload httpd

# Check status
systemctl status httpd

# Test configuration syntax
httpd -t

# List loaded modules
httpd -M

# Enable at boot
systemctl enable httpd
```

## Directory Structure

```
/etc/httpd/                   # Main config directory
├── conf/                     # Main config files
│   └── httpd.conf            # Main config file
├── conf.d/                   # Additional configurations
├── conf.modules.d/           # Module configurations
└── modules/                  # Module binaries

/var/www/html/                # Default web root
/var/log/httpd/               # Log files
```

## Basic Virtual Host

```apache
# /etc/httpd/conf.d/example.conf
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/example
    ErrorLog logs/example_error_log
    CustomLog logs/example_access_log combined
    
    <Directory /var/www/example>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

## IP-Based Virtual Hosts

```apache
<VirtualHost 192.168.1.31:80>
    DocumentRoot /var/www/html/site1/
    DirectoryIndex index.html
</VirtualHost>
```

## Port-Based Virtual Hosts

```apache
# Add to /etc/httpd/conf/httpd.conf or a file in conf.d/
Listen 8080
Listen 8443

# In conf.d/site-8080.conf
<VirtualHost 192.168.1.34:8080>
    DocumentRoot /var/www/html/site2/
    DirectoryIndex index.html
</VirtualHost>
```

## Domain-Based Virtual Hosts

```apache
<VirtualHost *:80>
    ServerName domain1.com
    ServerAlias www.domain1.com
    DocumentRoot /var/www/html/domain1
</VirtualHost>
```

## SSL/TLS Configuration

```bash
# Install mod_ssl
yum install mod_ssl

# Generate self-signed certificate
mkdir /opt/ssl
cd /opt/ssl
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365

# Copy certificates
cp cert.pem /etc/pki/tls/certs/
cp key.pem /etc/pki/tls/private/

# Open firewall port
firewall-cmd --permanent --add-port=443/tcp
firewall-cmd --reload
```

```apache
# In /etc/httpd/conf.d/ssl.conf
<VirtualHost *:443>
    ServerName example.com
    DocumentRoot /var/www/html/example
    
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/cert.pem
    SSLCertificateKeyFile /etc/pki/tls/private/key.pem
    
    # Strong SSL settings
    SSLProtocol all -SSLv2 -SSLv3 -TLSv1 -TLSv1.1
    SSLCipherSuite HIGH:!aNULL:!MD5:!3DES
    SSLHonorCipherOrder on
</VirtualHost>
```

## Password Protection

```bash
# Install htpasswd tool if not present
yum install httpd-tools

# Create password file
htpasswd -c /etc/httpd/.htpasswd username
```

```apache
# Directory password protection
<Directory "/var/www/protected">
    AuthType Basic
    AuthName "Restricted Content"
    AuthUserFile /etc/httpd/.htpasswd
    Require valid-user
</Directory>
```

## Directory Access Controls

```apache
# Disable directory listing
<Directory /var/www/html>
    Options -Indexes
</Directory>

# Enable for specific directory
<Directory /var/www/html/downloads>
    Options +Indexes
</Directory>

# IP restrictions (allow specific)
<Directory /var/www/restricted>
    Require ip 192.168.1.7 192.168.1.8
</Directory>

# IP restrictions (deny specific)
<Directory /var/www/restricted>
    <RequireAll>
        Require all granted
        Require not ip 192.168.1.8
    </RequireAll>
</Directory>
```

## HTTP Redirects

```bash
# Enable required modules (usually pre-loaded)
# Load modules if needed in /etc/httpd/conf.modules.d/00-proxy.conf
LoadModule rewrite_module modules/mod_rewrite.so
LoadModule alias_module modules/mod_alias.so
```

```apache
# Simple page redirect
Redirect /oldpage.html /newpage.html

# Permanent redirect to different domain
Redirect permanent /oldpage.html https://newdomain.com/newpage.html

# Redirect entire site
Redirect 301 / https://newdomain.com/

# HTTP to HTTPS redirect in VirtualHost
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    
    # Option 1: Simple redirect
    Redirect permanent / https://example.com/
    
    # Option 2: Using RewriteEngine
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
</VirtualHost>

# Non-www to www redirect
<VirtualHost *:80>
    ServerName example.com
    RewriteEngine On
    RewriteCond %{HTTP_HOST} ^example\.com$ [NC]
    RewriteRule ^(.*)$ https://www.example.com/$1 [R=301,L]
</VirtualHost>

# Redirect in .htaccess (requires AllowOverride All)
# /var/www/html/.htaccess
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
```

## User Directories

```apache
# Enable userdir in /etc/httpd/conf.d/userdir.conf
<IfModule mod_userdir.c>
    UserDir enabled
    UserDir public_html
</IfModule>

<Directory "/home/*/public_html">
    AllowOverride All
    Options MultiViews Indexes SymLinksIfOwnerMatch
    Require all granted
</Directory>

# Create user directory
mkdir /home/username/public_html
chmod 711 /home/username
chmod 755 /home/username/public_html
chown username:username /home/username/public_html

# Set SELinux context
semanage fcontext -a -t httpd_sys_content_t "/home/username/public_html(/.*)?"
restorecon -Rv /home/username/public_html
```

## WebDAV Configuration

```bash
# Install mod_dav
yum install mod_dav_fs

# Create directory
mkdir /var/www/html/webdav
chown apache:apache /var/www/html/webdav
```

```apache
# In conf.d/webdav.conf
<Directory /var/www/html/webdav>
    DAV On
    AuthType Basic
    AuthName "WebDAV"
    AuthUserFile /etc/httpd/.htpasswd
    Require valid-user
</Directory>
```

## CGI Scripts

```bash
# Add CGI configuration to httpd.conf or conf.d file
ScriptAlias /cgi-bin/ "/var/www/cgi-bin/"

# Create script directory
mkdir -p /var/www/cgi-bin
```

```apache
# In conf.d/cgi.conf
<Directory "/var/www/cgi-bin">
    Options +ExecCGI
    AddHandler cgi-script .cgi .pl .py .sh
    Require all granted
</Directory>
```

```bash
# Create sample CGI script
echo '#!/bin/bash
echo "Content-type: text/html"
echo
echo "<h1>Hello World</h1>"' > /var/www/cgi-bin/test.cgi

# Set permissions
chmod 755 /var/www/cgi-bin/test.cgi
chown apache:apache /var/www/cgi-bin/test.cgi

# Set SELinux context
semanage fcontext -a -t httpd_sys_script_exec_t "/var/www/cgi-bin(/.*)?"
restorecon -Rv /var/www/cgi-bin
```

## PHP Configuration

```bash
# Install PHP
yum install php php-mysqlnd

# For PHP-FPM (recommended)
yum install php-fpm
systemctl enable --now php-fpm

# Edit /etc/httpd/conf.d/php.conf to use PHP-FPM
```

## SELinux Configuration

```bash
# Check SELinux status
getenforce

# Set SELinux context for web content
semanage fcontext -a -t httpd_sys_content_t "/var/www/html(/.*)?"
restorecon -Rv /var/www/html

# Allow httpd to connect to network (for proxies)
setsebool -P httpd_can_network_connect 1

# Allow httpd to connect to databases
setsebool -P httpd_can_network_connect_db 1

# Allow httpd to send email
setsebool -P httpd_can_sendmail 1
```

## WordPress Quick Setup

```bash
# Create database
mysql -u root -p
CREATE DATABASE wordpress;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;

# Download and configure WordPress
cd /var/www/html/
wget https://wordpress.org/latest.zip
yum install unzip
unzip latest.zip
chown -R apache:apache wordpress/

# Set SELinux context
semanage fcontext -a -t httpd_sys_content_t "/var/www/html/wordpress(/.*)?"
semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/html/wordpress/wp-content(/.*)?"
restorecon -Rv /var/www/html/wordpress
```

## Common Troubleshooting

```bash
# Check error logs
tail -f /var/log/httpd/error_log

# Check access logs
tail -f /var/log/httpd/access_log

# Check open ports
netstat -tuln | grep httpd

# Test page access
curl -I http://localhost

# SELinux issues
grep httpd /var/log/audit/audit.log
ausearch -c httpd --raw

# Debugging SSL
openssl s_client -connect example.com:443
```
