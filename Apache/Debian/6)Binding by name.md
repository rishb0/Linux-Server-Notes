# Binding with Domain Names in Apache
Apache can bind to specific domain names using the `ServerName` and `ServerAlias` directives in Virtual Hosts. This allows you to host multiple websites on the same IP address, each accessible by a unique domain name.

## 1. Configure Hostnames (DNS or /etc/hosts)
To simulate DNS resolution, define the domain names in the `/etc/hosts` file.

Edit `/etc/hosts`:
```bash
vim /etc/hosts
```
Add the following entries:
```
127.0.0.1       localhost ns1 ns1.armour.local
::1             localhost localhost6 ns1.armour.local
192.168.1.31    ns1.armour.local armour.local ai.local www.ai.local infosec.local www.infosec.local
```
This will resolve the domain names to the local IP (192.168.1.31).

## 2. Test DNS Resolution
Use `dig` to confirm that the domain names resolve to the correct IP:
```bash
dig www.armour.local +short
```
Example output:
```
192.168.1.31
```
Repeat for other domains:
```
dig www.infosec.local +short
```

```
dig www.ai.local +short
```

## 3. Create Apache Virtual Host Files
Define virtual hosts for each domain. Apache will route incoming requests based on the `ServerName` and `ServerAlias`.

### 3.1 Main Virtual Host (Default)
Create a default virtual host file:
```bash
vim /etc/apache2/sites-available/000-default.conf
```
Example:
```xml
<VirtualHost 192.168.1.31:80>
    DocumentRoot /var/www/html/armour/
    DirectoryIndex index.html
</VirtualHost>
```

### 3.2 Virtual Host for armour.local
Create a virtual host file for `armour.local`:
```bash
vim /etc/apache2/sites-available/armour.conf
```
Example:
```xml
<VirtualHost 192.168.1.31:80>
    ServerName armour.local
    ServerAlias www.armour.local
    DocumentRoot /var/www/html/site1/
    DirectoryIndex index.html
</VirtualHost>
```

### 3.3 Virtual Host for infosec.local
Create a virtual host file for `infosec.local`:
```bash
vim /etc/apache2/sites-available/infosec.conf
```
Example:
```xml
<VirtualHost 192.168.1.31:80>
    ServerName infosec.local
    ServerAlias www.infosec.local
    DocumentRoot /var/www/html/site2/
    DirectoryIndex index.html
</VirtualHost>
```

### 3.4 Virtual Host for ai.local
Create a virtual host file for `ai.local`:
```bash
vim /etc/apache2/sites-available/ai.conf
```
Example:
```xml
<VirtualHost 192.168.1.34:80>
    ServerName ai.local
    ServerAlias www.ai.local
    DocumentRoot /var/www/html/site3/
    DirectoryIndex index.html
</VirtualHost>
```

### 3.5 Enable the Virtual Host Files
Enable each configuration:
```bash
a2ensite armour.conf
a2ensite infosec.conf
a2ensite ai.conf
```

## 4. Binding Domains with Different Ports
You can also bind the same domains to different ports using `Listen` in `/etc/apache2/ports.conf`.

First, edit the ports.conf file:
```bash
vim /etc/apache2/ports.conf
```
Add the following lines:
```
Listen 80
Listen 8080
Listen 8081
```

### 4.1 Bind armour.local on Port 8080
Create a virtual host file:
```bash
vim /etc/apache2/sites-available/armour-8080.conf
```
Example:
```xml
<VirtualHost 192.168.1.31:8080>
    ServerName armour.local
    ServerAlias www.armour.local
    DocumentRoot /var/www/html/site2/
    DirectoryIndex index.html
</VirtualHost>
```

### 4.2 Bind infosec.local on Port 8080
Create a virtual host file for `infosec.local` on port 8080:
```bash
vim /etc/apache2/sites-available/infosec-8080.conf
```
Example:
```xml
<VirtualHost 192.168.1.31:8080>
    ServerName infosec.local
    ServerAlias www.infosec.local
    DocumentRoot /var/www/html/site3/
    DirectoryIndex index.html
</VirtualHost>
```

### 4.3 Bind ai.local on Port 8081
Create a virtual host file for `ai.local` on port 8081:
```bash
vim /etc/apache2/sites-available/ai-8081.conf
```
Example:
```xml
<VirtualHost 192.168.1.31:8081>
    ServerName ai.local
    ServerAlias www.ai.local
    DocumentRoot /var/www/html/site4/
    DirectoryIndex index.html
</VirtualHost>
```

### 4.4 Enable the Additional Virtual Host Files
```bash
a2ensite armour-8080.conf
a2ensite infosec-8080.conf
a2ensite ai-8081.conf
```

## 5. Check Apache Configuration
Validate the Apache configuration:
```bash
apache2ctl -t
```
Example output:
```
Syntax OK
```

## 6. Configure Firewall
Allow the required ports through the firewall:
```bash
ufw allow 80/tcp
ufw allow 8080/tcp
ufw allow 8081/tcp
```

## 7. Restart Apache
Apply the changes:
```bash
systemctl restart apache2.service
```

## 8. Confirm Ports and Bindings
Use `netstat` or `ss` to check if Apache is listening on the expected ports:
```bash
netstat -nltup | grep apache2
```
Example output:
```
tcp        0      0 192.168.1.31:80        0.0.0.0:*          LISTEN      1234/apache2
tcp        0      0 192.168.1.31:8080      0.0.0.0:*          LISTEN      1234/apache2
tcp        0      0 192.168.1.31:8081      0.0.0.0:*          LISTEN      1234/apache2
```

## 9. Test in Browser
You should be able to access the sites by their domain names:

| Domain                    | Port | Expected Output |
| ------------------------- | ---- | --------------- |
| http://armour.local       | 80   | site2 homepage  |
| http://www.armour.local   | 80   | site2 homepage  |
| http://infosec.local      | 80   | site3 homepage  |
| http://www.infosec.local  | 80   | site3 homepage  |
| http://ai.local           | 80   | site4 homepage  |
| http://www.ai.local       | 80   | site4 homepage  |
| http://armour.local:8080  | 8080 | site2 homepage  |
| http://infosec.local:8080 | 8080 | site3 homepage  |
| http://ai.local:8081      | 8081 | site4 homepage  |

## Example: Complete Apache Virtual Host Configuration
Example of a complete Apache configuration using different domain names and ports:

```xml
# In /etc/apache2/ports.conf:
Listen 80
Listen 8080
Listen 8081

# In individual site configuration files:
<VirtualHost 192.168.1.31:80>
    ServerName armour.local
    ServerAlias www.armour.local
    DocumentRoot /var/www/html/site2/
    DirectoryIndex index.html
</VirtualHost>

<VirtualHost 192.168.1.31:80>
    ServerName infosec.local
    ServerAlias www.infosec.local
    DocumentRoot /var/www/html/site3/
    DirectoryIndex index.html
</VirtualHost>

<VirtualHost 192.168.1.31:8080>
    ServerName armour.local
    ServerAlias www.armour.local
    DocumentRoot /var/www/html/site2/
    DirectoryIndex index.html
</VirtualHost>

<VirtualHost 192.168.1.31:8081>
    ServerName ai.local
    ServerAlias www.ai.local
    DocumentRoot /var/www/html/site4/
    DirectoryIndex index.html
</VirtualHost>
```

✅ Apache domain-based virtual host binding is now complete! 😎