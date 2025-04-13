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
rpm -qa | grep httpd
```

This command checks if the Apache HTTP Server is already installed.

### Install Apache (if not installed)

```bash
yum install httpd*
```

Installs Apache and all associated packages on RHEL/CentOS.

### List Loaded Apache Modules

```bash
httpd -M
```

Displays all currently enabled Apache modules.

### Verify WebDAV Modules

```bash
httpd -M | grep fs
```

Expected output should include:

```
dav_fs_module (shared)
```

Indicates `mod_dav_fs` is enabled for WebDAV functionality.

## 2. Create and Set Up WebDAV Directory

### Create WebDAV Directory

```bash
mkdir /var/www/html/webdav
```

Creates the directory for WebDAV file storage.

### Set Ownership to Apache User

```bash
chown apache:apache /var/www/html/webdav/
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

### Edit Apache Main Configuration

```bash
vim /etc/httpd/conf/httpd.conf
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
## 5. Configure SELinux and Firewall

### Restart Apache to Apply Changes

```bash
systemctl restart httpd.service
```
or
```bash
systemctl restart httpd
```
### Check SELinux Status
```bash
sestatus
```
### Disable SELinux (for testing only)
```bash
vim /etc/sysconfig/selinux
```

Set this line:

```
SELINUX=disabled
```

Recommended to use context labeling instead of disabling SELinux.

## 6. Secure WebDAV with Authentication

### Create .htpasswd File and Users

```bash
htpasswd -c /etc/httpd/htpasswd dev
htpasswd /etc/httpd/htpasswd admin
htpasswd /etc/httpd/htpasswd root
htpasswd /etc/httpd/htpasswd user
```

Creates the password file and adds users for basic authentication.

### View Password File

```bash
cat /etc/httpd/htpasswd
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

### Edit Apache Configuration Again

```bash
vim /etc/httpd/conf/httpd.conf
```

### Add Authenticated WebDAV Block

```apache
IncludeOptional conf.d/*.conf

<VirtualHost *:80>
    DocumentRoot /var/www/html/
    DirectoryIndex index.html
    <Directory /var/www/html/webdav>
        DAV On
        AuthType Basic
        AuthName "webdav"
        AuthUserFile /etc/httpd/htpasswd
        Require valid-user
    </Directory>
</VirtualHost>
```

## 8. Restart Apache After Changes

```bash
apachectl restart
```
or
```bash
systemctl restart httpd
```

## Notes for Production

* Always use HTTPS to protect credentials during transmission.
* Consider limiting access by IP or using firewalls.
* Use SELinux context labeling instead of disabling it:

```bash
chcon -R -t httpd_sys_content_t /var/www/html/webdav
```
## 8. Restart Apache After Changes

```bash
apachectl restart
```

or

```bash
systemctl restart httpd
```

## Notes for Production

* Always use HTTPS to protect credentials during transmission.
* Consider limiting access by IP or using firewalls.
* Use SELinux context labeling instead of disabling it:

```bash
chcon -R -t httpd_sys_content_t /var/www/html/webdav
```
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
curl -X DELETE http://192.168.1.22/webdav/phpinfo.php
```

```bash
curl -v -X DELETE http://192.168.1.22/webdav/1.txt
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
```
yum install cadaver        # CentOS/RHEL
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




