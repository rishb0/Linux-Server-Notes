# Binding with Port Number in Apache
Apache can bind to specific ports, allowing you to run multiple websites on different ports using Virtual Hosts. This is useful when you want to test different sites on the same IP but using unique ports.

## 1. Modify Apache Configuration
Open the main Apache configuration file:
```bash
vim /etc/httpd/conf/httpd.conf
```

### Add Listen Directives for New Ports
- The `Listen` directive tells Apache which ports to bind to.
- Add the following lines to allow Apache to listen on ports 80, 8080, 8081, 8082:
```
Listen 80
Listen 8080
Listen 8081
Listen 8082
```
Save and exit.

## 2. Create Virtual Host Files
Create separate virtual host files for each port under `/etc/httpd/conf.d/`.

### 2.1 Default HTTP Port (80)
Create a virtual host for port 80:
```bash
vim /etc/httpd/conf.d/site1.conf
```
Example:
```xml
<VirtualHost 192.168.1.34:80>
    DocumentRoot /var/www/html/site1/
    DirectoryIndex index.html
</VirtualHost>
```

### 2.2 Virtual Host for Port 8080
Create a virtual host for port 8080:
```bash
vim /etc/httpd/conf.d/site2.conf
```
Example:
```xml
Listen 8080
<VirtualHost 192.168.1.34:8080>
    DocumentRoot /var/www/html/site2/
    DirectoryIndex index.html
</VirtualHost>
```

### 2.3 Virtual Host for Port 8081
Create a virtual host for port 8081:
```bash
vim /etc/httpd/conf.d/site3.conf
```
Example:
```xml
Listen 8081
<VirtualHost 192.168.1.34:8081>
    DocumentRoot /var/www/html/site3/
    DirectoryIndex index.html
</VirtualHost>
```

### 2.4 Virtual Host for Port 8082
Create a virtual host for port 8082:
```bash
vim /etc/httpd/conf.d/site4.conf
```
Example:
```xml
Listen 8082
<VirtualHost 192.168.1.34:8082>
    DocumentRoot /var/www/html/site4/
    DirectoryIndex index.html
</VirtualHost>
```

### 2.5 Virtual Host for Alternate IP and Port
If you want to bind to another IP (192.168.1.50) and a custom port (81, 82):

#### Port 81:
```bash
vim /etc/httpd/conf.d/site5.conf
```
Example:
```xml
Listen 81
<VirtualHost 192.168.1.50:81>
    DocumentRoot /var/www/html/site5/
    DirectoryIndex index.html
</VirtualHost>
```

#### Port 82:
```bash
vim /etc/httpd/conf.d/site6.conf
```
Example:
```xml
Listen 82
<VirtualHost 192.168.1.50:82>
    DocumentRoot /var/www/html/site6/
    DirectoryIndex index.html
</VirtualHost>
```
## 3. Configure Firewall
```bash
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --permanent --add-port=8081/tcp
firewall-cmd --permanent --add-port=8082/tcp
firewall-cmd --reload
```

## 4. Validate Apache Configuration
Check if Apache configuration is valid:
```bash
httpd -t
```
Example output:
```
Syntax OK
```

## 5. Restart Apache
Apply the changes by restarting Apache:
```bash
systemctl restart httpd.service
```
## 6. Verify Listening Ports
Use `netstat` to confirm Apache is listening on the correct ports:
```bash
netstat -nl tup | grep httpd
```
Example output:
```
tcp        0      0 192.168.1.34:80        0.0.0.0:*          LISTEN      1234/httpd
tcp        0      0 192.168.1.34:8080      0.0.0.0:*          LISTEN      1234/httpd
tcp        0      0 192.168.1.34:8081      0.0.0.0:*          LISTEN      1234/httpd
tcp        0      0 192.168.1.34:8082      0.0.0.0:*          LISTEN      1234/httpd
tcp        0      0 192.168.1.50:81        0.0.0.0:*          LISTEN      1234/httpd
tcp        0      0 192.168.1.50:82        0.0.0.0:*          LISTEN      1234/httpd
```

## Example: Full Virtual Host Configuration
Example of a complete Apache virtual host configuration using different ports:
```xml
Listen 80
Listen 8080
Listen 8081
Listen 8082
Listen 81
Listen 82

<VirtualHost 192.168.1.31:80>
    DocumentRoot /var/www/html/site1/
    DirectoryIndex index.html
</VirtualHost>

<VirtualHost 192.168.1.31:8080>
    DocumentRoot /var/www/html/site2/
    DirectoryIndex index.html
</VirtualHost>

<VirtualHost 192.168.1.31:8081>
    DocumentRoot /var/www/html/site3/
    DirectoryIndex index.html
</VirtualHost>

<VirtualHost 192.168.1.34:8082>
    DocumentRoot /var/www/html/site4/
    DirectoryIndex index.html
</VirtualHost>

<VirtualHost 192.168.1.50:81>
    DocumentRoot /var/www/html/site5/
    DirectoryIndex index.html
</VirtualHost>

<VirtualHost 192.168.1.50:82>
    DocumentRoot /var/www/html/site6/
    DirectoryIndex index.html
</VirtualHost>
```
## Summary
| Port | Virtual Host Configuration         | Purpose        |
|------|-----------------------------------|-----------------|
| 80   | `<VirtualHost 192.168.1.31:80>`   | Main site      |
| 8080 | `<VirtualHost 192.168.1.31:8080>` | Test site 1    |
| 8081 | `<VirtualHost 192.168.1.31:8081>` | Test site 2    |
| 8082 | `<VirtualHost 192.168.1.31:8082>` | Test site 3    |
| 81   | `<VirtualHost 192.168.1.50:81>`   | Alternate site 1|
| 82   | `<VirtualHost 192.168.1.50:82>`   | Alternate site 2|

🚀 Apache virtual host binding with ports is now complete!
