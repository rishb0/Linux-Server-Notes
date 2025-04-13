# WebDAV-with-Apache

## WebDAV with Apache

WebDAV (Web Distributed Authoring and Versioning) extends the HTTP protocol to allow users to collaboratively edit and manage files on a remote web server. Apache HTTP Server supports WebDAV using the `mod_dav` and `mod_dav_fs` modules.

## Features of WebDAV

* Remote file management via HTTP/S.
* Supports file creation, upload, editing, and deletion.
* Useful for collaborative development and web file hosting.
* Easily integrates with tools like SVN and Git.
* Clients can mount WebDAV shares as network drives or local filesystems.

## 1. Install and Verify Apache

### Check if Apache is Installed

```bash
dpkg -l | grep apache2
```

This command checks if the Apache HTTP Server is already installed.

### Install Apache (if not installed)

```bash
apt update
apt install apache2
```

Installs Apache and all associated packages on Debian/Ubuntu.

### List Loaded Apache Modules

```bash
apache2ctl -M
```

Displays all currently enabled Apache modules.

### Enable WebDAV Modules

```bash
a2enmod dav
a2enmod dav_fs
```

Enables the WebDAV modules in Apache.

### Verify WebDAV Modules

```bash
apache2ctl -M | grep dav
```

Expected output should include:

```
dav_module (shared)
dav_fs_module (shared)
```

Indicates `mod_dav` and `mod_dav_fs` are enabled for WebDAV functionality.

## 2. Create and Set Up WebDAV Directory

### Create WebDAV Directory

```bash
mkdir /var/www/html/webdav
```

Creates the directory for WebDAV file storage.

### Set Ownership to Apache User

```bash
chown www-data:www-data /var/www/html/webdav/
```

Ensures Apache has ownership of the directory.

### Set Permissions

```bash
chmod 777 /var/www/html/webdav/
```

For production environments, use `755` or `775` with tighter controls.

## 3. Test WebDAV Access

### Send OPTIONS Request to WebDAV Directory

```bash
curl -v -X OPTIONS http://192.168.1.33/webdav/
```

Checks available HTTP methods supported by the WebDAV-enabled directory.


## 4. Configure WebDAV in Apache

### Create WebDAV Configuration File

```bash
vim /etc/apache2/sites-available/webdav.conf
```

### Add WebDAV Virtual Host

```apache
<VirtualHost *:80>
    DocumentRoot /var/www/html/
    DirectoryIndex index.html
    <Directory /var/www/html/webdav>
        DAV On
    </Directory>
</VirtualHost>
```

This enables WebDAV functionality for the `/webdav` directory on port 80.

### Enable the Site

```bash
a2ensite webdav.conf
```

## 5. Configure AppArmor and Firewall

### Restart Apache to Apply Changes

```bash
systemctl restart apache2.service
```
or
```bash
systemctl restart apache2
```

### Check AppArmor Status (Debian's security system)
```bash
aa-status
```

### Configure Firewall if Needed
```bash
ufw allow 80/tcp
```

## 6. Secure WebDAV with Authentication

### Create .htpasswd File and Users

```bash
htpasswd -c /etc/apache2/htpasswd dev
htpasswd /etc/apache2/htpasswd admin
htpasswd /etc/apache2/htpasswd root
htpasswd /etc/apache2/htpasswd user
```

Creates the password file and adds users for basic authentication.

### View Password File

```bash
cat /etc/apache2/htpasswd
```

Displays encrypted passwords.

Example output:

```
dev:$apr1$SGXmFNJ4$fhBza.Fu25P1SMuncBcATuG/
admin:$apr1$8MImNcn0$EpS5Lhk21/HVjXV7mKkpxs4C/
root:$apr1$Qey...Fan$SyZGIMsUP13DDMrnWHgOnlU.
user:$apr1$z2p/.J6/mB$v.zh.doWHBq/34CABJmwkX/
```

## 7. Enable Authentication in Apache Configuration

### Edit WebDAV Configuration

```bash
vim /etc/apache2/sites-available/webdav.conf
```

### Add Authenticated WebDAV Block

```apache
<VirtualHost *:80>
    DocumentRoot /var/www/html/
    DirectoryIndex index.html
    <Directory /var/www/html/webdav>
        DAV On
        AuthType Basic
        AuthName "webdav"
        AuthUserFile /etc/apache2/htpasswd
        Require valid-user
    </Directory>
</VirtualHost>
```

## 8. Restart Apache After Changes

```bash
apache2ctl restart
```
or
```bash
systemctl restart apache2
```

## Notes for Production

* Always use HTTPS to protect credentials during transmission.
* Consider limiting access by IP or using firewalls.
* If using AppArmor, check profiles if permission issues arise.

## WebDAV Client Testing

**Basic OPTIONS Request**
```
curl -v -X OPTIONS http://192.168.1.36/webdav/
```
```
curl -i -k -X OPTIONS https://192.168.1.36/webdav/
```

**Scan HTTP Methods via Nmap**
```
nmap -v -sT -sV -A -O -p 80 --script=http-methods.nse --script-args http-methods.url-path='/webdav/' 192.168.1.36
```

### File Upload & Deletion with cURL

#### Upload a File

```bash
curl -X PUT -d "hi" http://192.168.1.36/webdav/1.txt
```

#### Upload a PHP File (for testing)

```bash
curl -X PUT -d "<?php phpinfo(); ?>" http://192.168.1.36/webdav/phpinfo.php
```

#### Delete Uploaded File

```bash
curl -X DELETE http://192.168.1.36/webdav/phpinfo.php
```

```bash
curl -v -X DELETE http://192.168.1.36/webdav/1.txt
```

### Authenticated Requests

```bash
curl -v -u dev:123 -X OPTIONS http://192.168.1.36/test/
```
```
curl -u dev:123 -X PUT -d "hi" http://192.168.1.36/test/3.txt
```
```
curl -X PUT -u user1:123 -d "<?php phpinfo(); ?>" http://192.168.1.36/webdav/phpinfo.php
```
```
curl -u user1:123 -X DELETE http://192.168.1.36/webdav/phpinfo.php
```
```
curl -v -u root:root -X PUT -d "hi" http://192.168.1.36/webdav/1.txt
```

### Using `cadaver` CLI WebDAV Client

#### Install cadaver
```bash
apt install cadaver        # Debian/Ubuntu
```

#### Connect to WebDAV Server
```bash
cadaver http://192.168.1.36/webdav/
```

#### Sample cadaver Session
```
dav:/webdav/> ?
dav:/webdav/> ls
dav:/webdav/> put network.png
dav:/webdav/> mkdir newdir
dav:/webdav/> mput *.php
dav:/webdav/> delete *
```

#### Connect to Another Location
```bash
cadaver http://192.168.1.36/test/
```