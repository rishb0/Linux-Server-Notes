# HTTP to HTTPS Redirect in Apache (Debian)

## Prerequisites

First, ensure required Apache modules are enabled:

```bash
a2enmod ssl
a2enmod rewrite
systemctl restart apache2
```

## Method 1: VirtualHost Configuration (Recommended)

Edit your site configuration in `/etc/apache2/sites-available/`:

```bash
vim /etc/apache2/sites-available/example.conf
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
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

# HTTPS VirtualHost - serves content
<VirtualHost *:443>
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/html/example
    
    SSLEngine on
    SSLCertificateFile /etc/apache2/ssl/example.crt
    SSLCertificateKeyFile /etc/apache2/ssl/example.key
    
    # Other SSL options as needed
    # SSLCertificateChainFile /etc/apache2/ssl/example-chain.crt
    
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
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

## Method 3: Site-Wide Redirect

For redirecting all HTTP traffic to HTTPS across all sites:

```bash
vim /etc/apache2/sites-available/000-default.conf
```

Add to the VirtualHost:

```apache
<VirtualHost *:80>
    # ... existing configuration ...
    
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
</VirtualHost>
```

## Apply Changes

```bash
# Test configuration
apache2ctl configtest

# If "Syntax OK", reload Apache
systemctl reload apache2
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

- Check Apache error logs: `tail -f /var/log/apache2/error.log`
- Verify mod_rewrite is enabled: `apache2ctl -M | grep rewrite`
- Ensure proper permissions on .htaccess file if used
- Test with browser cache disabled