# HTTP to HTTPS Redirect in Apache (CentOS)

## Prerequisites

First, ensure required Apache modules are loaded:

```bash
# Check if the modules are already loaded
httpd -M | grep ssl
httpd -M | grep rewrite

# If not loaded, enable them in configuration
echo "LoadModule ssl_module modules/mod_ssl.so" > /etc/httpd/conf.modules.d/00-ssl.conf
echo "LoadModule rewrite_module modules/mod_rewrite.so" > /etc/httpd/conf.modules.d/00-rewrite.conf

# Install mod_ssl if not already installed
yum install mod_ssl

# Restart Apache to load modules
systemctl restart httpd
```

## Method 1: VirtualHost Configuration (Recommended)

Edit your site configuration:

```bash
vim /etc/httpd/conf.d/example.conf
```

Add HTTP to HTTPS redirect:

```apache
# HTTP VirtualHost - redirects to HTTPS
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    
    # Option 1: Simple redirect
    Redirect permanent / https://example.com/
    
    # Option 2: Using RewriteEngine (more flexible)
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
    
    # Logs
    ErrorLog logs/example-error_log
    CustomLog logs/example-access_log combined
</VirtualHost>

# HTTPS VirtualHost - serves content
<VirtualHost *:443>
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/html/example
    
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/example.crt
    SSLCertificateKeyFile /etc/pki/tls/private/example.key
    
    # Other SSL options as needed
    # SSLCertificateChainFile /etc/pki/tls/certs/example-chain.crt
    
    ErrorLog logs/example-ssl-error_log
    CustomLog logs/example-ssl-access_log combined
</VirtualHost>
```

## Method 2: Using .htaccess

If you have `AllowOverride All` set in your Apache configuration, create/edit `.htaccess` in your document root:

```bash
vim /var/www/html/example/.htaccess
```

Add these lines:

```apache
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
```

Ensure .htaccess is allowed in httpd.conf:

```apache
<Directory "/var/www/html">
    # ...
    AllowOverride All
    # ...
</Directory>
```

## Method 3: Site-Wide Redirect

For redirecting all HTTP traffic to HTTPS across all sites:

```bash
vim /etc/httpd/conf.d/ssl.conf
```

Add near the top:

```apache
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
</IfModule>
```

## Apply Changes

```bash
# Test configuration
httpd -t

# If "Syntax OK", restart Apache
systemctl restart httpd
```

## SELinux Considerations

CentOS uses SELinux by default, which might block redirects:

```bash
# Allow httpd to connect to network
setsebool -P httpd_can_network_connect 1

# If using custom directories, update context
semanage fcontext -a -t httpd_sys_content_t "/path/to/custom/directory(/.*)?"
restorecon -Rv /path/to/custom/directory
```

## Testing

1. Clear browser cache or use private/incognito mode
2. Visit http://example.com/ - should redirect to https://example.com/
3. Check using curl:
   ```bash
   curl -I http://example.com/
   ```
   Should show `301 Moved Permanently` and `Location: https://example.com/`

## Troubleshooting

- Check Apache error logs: `tail -f /var/log/httpd/error_log`
- Verify mod_rewrite is loaded: `httpd -M | grep rewrite`
- Check SELinux logs if experiencing permissions issues: `grep httpd /var/log/audit/audit.log`
- Test with browser cache disabled
- Ensure firewall allows HTTPS traffic: `firewall-cmd --list-all`