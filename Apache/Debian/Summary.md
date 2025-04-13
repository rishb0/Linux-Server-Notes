# Apache Quick Reference Guide for Debian

A concise reference for common Apache configurations on Debian systems.

## Installation & Basic Commands

```bash
# Install Apache
apt update
apt install apache2

# Start, stop, restart
systemctl start apache2
systemctl stop apache2
systemctl restart apache2
systemctl reload apache2

# Check status
systemctl status apache2

# Test configuration syntax
apache2ctl configtest

# List enabled modules
apache2ctl -M

# Enable/disable modules
a2enmod module_name
a2dismod module_name

# Enable/disable sites
a2ensite site_name.conf
a2dissite site_name.conf

# Enable/disable configs
a2enconf config_name
a2disconf config_name
```

## Directory Structure

```
/etc/apache2/                  # Main config directory
├── apache2.conf               # Main config file
├── ports.conf                 # Port definitions
├── mods-available/            # Available modules
├── mods-enabled/              # Enabled modules (symlinks)
├── sites-available/           # Available virtual hosts
├── sites-enabled/             # Enabled virtual hosts (symlinks)
├── conf-available/            # Available configurations
└── conf-enabled/              # Enabled configurations (symlinks)

/var/www/html/                 # Default web root
/var/log/apache2/              # Log files
```

## Basic Virtual Host

```apache
# /etc/apache2/sites-available/example.conf
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/example
    ErrorLog ${APACHE_LOG_DIR}/example_error.log
    CustomLog ${APACHE_LOG_DIR}/example_access.log combined
    
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

```bash
# Add to /etc/apache2/ports.conf
Listen 8080
Listen 8443
```

```apache
# In sites-available/site-8080.conf
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
# Enable SSL module
a2enmod ssl

# Generate self-signed certificate
mkdir /opt/ssl
cd /opt/ssl
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365

# Copy certificates
cp cert.pem /etc/ssl/certs/
cp key.pem /etc/ssl/private/

# Open firewall port
ufw allow 443/tcp
```

```apache
# In sites-available/site-ssl.conf
<VirtualHost *:443>
    ServerName example.com
    DocumentRoot /var/www/html/example
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/cert.pem
    SSLCertificateKeyFile /etc/ssl/private/key.pem
    
    # Strong SSL settings
    SSLProtocol all -SSLv2 -SSLv3 -TLSv1 -TLSv1.1
    SSLCipherSuite HIGH:!aNULL:!MD5:!3DES
    SSLHonorCipherOrder on
</VirtualHost>
```

## Password Protection

```bash
# Create password file
htpasswd -c /etc/apache2/htpasswd username
```

```apache
# Directory password protection
<Directory "/var/www/protected">
    AuthType Basic
    AuthName "Restricted Content"
    AuthUserFile /etc/apache2/htpasswd
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

## User Directories

```bash
# Enable userdir module
a2enmod userdir

# Configure in /etc/apache2/mods-available/userdir.conf
<IfModule mod_userdir.c>
    UserDir enabled
    UserDir public_html
</IfModule>

<Directory "/home/*/public_html">
    AllowOverride All
    Options MultiViews Indexes SymLinksIfOwnerMatch
    Require method GET POST OPTIONS
</Directory>

# Create user directory
mkdir /home/username/public_html
chmod 711 /home/username
chmod 755 /home/username/public_html
chown username:username /home/username/public_html
```
## HTTP Redirects

```bash
# Enable required modules
a2enmod rewrite
a2enmod alias
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

# www to non-www redirect
<VirtualHost *:80>
    ServerName www.example.com
    RewriteEngine On
    RewriteRule ^(.*)$ https://example.com/$1 [R=301,L]
</VirtualHost>

# Redirect in .htaccess (requires AllowOverride All)
# /var/www/html/.htaccess
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]

# Redirect specific directory
RedirectMatch 301 ^/olddir/(.*)$ /newdir/$1

# Redirect with query parameters preserved
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteCond %{HTTP_HOST} ^example\.com$
RewriteRule (.*) https://www.example.com/$1 [R=301,L,QSA]
```
## WebDAV Configuration

```bash
# Enable modules
a2enmod dav
a2enmod dav_fs

# Create directory
mkdir /var/www/html/webdav
chown www-data:www-data /var/www/html/webdav
```

```apache
# In sites-available/webdav.conf
<Directory /var/www/html/webdav>
    DAV On
    AuthType Basic
    AuthName "WebDAV"
    AuthUserFile /etc/apache2/htpasswd
    Require valid-user
</Directory>
```

## CGI Scripts

```bash
# Enable CGI module
a2enmod cgi

# Create script directory
mkdir -p /usr/lib/cgi-bin
```

```apache
# In conf-available/cgi-enabled.conf
<Directory "/usr/lib/cgi-bin">
    Options +ExecCGI
    AddHandler cgi-script .cgi .pl .py .sh
    Require all granted
</Directory>

# ScriptAlias (automatic CGI execution)
ScriptAlias /cgi-bin/ "/usr/lib/cgi-bin/"
```

```bash
# Create sample CGI script
echo '#!/bin/bash
echo "Content-type: text/html"
echo
echo "<h1>Hello World</h1>"' > /usr/lib/cgi-bin/test.cgi

# Set permissions
chmod 755 /usr/lib/cgi-bin/test.cgi
chown www-data:www-data /usr/lib/cgi-bin/test.cgi
```

## PHP Configuration

```bash
# Install PHP
apt install php php-common php-mysql

# For PHP-FPM (recommended)
apt install php-fpm
a2enmod proxy_fcgi setenvif
a2enconf php7.4-fpm  # Replace with your PHP version
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
apt install unzip
unzip latest.zip
chown -R www-data:www-data wordpress/
```

## Common Troubleshooting

```bash
# Check error logs
tail -f /var/log/apache2/error.log

# Check access logs
tail -f /var/log/apache2/access.log

# Check open ports
netstat -tuln | grep apache2

# Test page access
curl -I http://localhost

# Debugging SSL
openssl s_client -connect example.com:443
```
