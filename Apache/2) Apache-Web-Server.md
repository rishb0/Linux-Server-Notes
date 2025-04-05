# Apache Web Server Setup and Configuration

This document outlines the installation, configuration, and management of the Apache Web Server on a Linux-based system using `yum` and `rpm` package managers. It includes essential commands, configuration examples, and troubleshooting steps.

---

## 1. Check If Apache is Installed

Use the following command to check if Apache (`httpd`) is already installed:

```bash
rpm -qa | grep httpd
```

---

## 2. Install Apache Web Server

Install Apache using `yum`:

```bash
yum install httpd
```

Install all related Apache packages:

```bash
yum install httpd*
```

---

## 3. Verify Apache Installation

- **Check package information:**

  ```bash
  rpm -qi httpd
  ```

- **List installed files:**

  ```bash
  rpm -ql httpd
  ```

  > **Note:**  
  > The main configuration file `/etc/httpd/conf/httpd.conf` is not listed in `rpm -ql httpd` because it is not installed directly from the `httpd` package. Instead, it is usually part of `httpd-filesystem` or created during the first run of Apache. You can check it with:
  >
  > ```bash
  > rpm -ql httpd-filesystem
  > ```
  > or
  > ```bash
  > rpm -qc httpd-filesystem
  > ```

- **List configuration files:**

  ```bash
  rpm -qc httpd
  ```

- **List documentation files:**

  ```bash
  rpm -qd httpd
  ```

---

## 4. Apache Configuration

### View Apache Configuration

- **Check the main configuration file:**

  ```bash
  tail /etc/httpd/conf/httpd.conf
  ```

- **Find the `conf.modules.d` directive:**

  ```bash
  grep conf\.modules\.d /etc/httpd/conf/httpd.conf
  ```

  Then list the content of the modules directory:

  ```bash
  ls /etc/httpd/conf.modules.d/
  ```

  **Example Output:**

  ```
  00-base.conf    00-lua.conf       00-proxy.conf    10-h2.conf
  00-brotli.conf  00-mpm.conf       00-systemd.conf  10-proxy_h2.conf
  00-dav.conf     00-optional.conf  01-cgi.conf      README
  ```

  These are Apache module configuration files enabling or configuring specific Apache modules (e.g., `mod_proxy`, `mod_mpm`, `mod_http2`).

- **Find the `conf.d` directive:**

  ```bash
  grep conf\.d /etc/httpd/conf/httpd.conf
  ```

  Then list the content of the configuration directory:

  ```bash
  ls /etc/httpd/conf.d/
  ```

  **Example Output:**

  ```
  autoindex.conf  manual.conf  README  userdir.conf  welcome.conf
  ```

  These files are used to configure specific features and virtual hosts. For instance, `autoindex.conf` controls directory listing and `welcome.conf` defines the default welcome page.

---

## 5. Manage Apache Service

### Check Apache Status

Check if the Apache service is running:

```bash
systemctl status httpd.service
```

### Start Apache Service

Start the Apache service:

```bash
systemctl start httpd.service
```

### Enable Apache to Start on Boot

Enable Apache to start automatically on system boot:

```bash
systemctl enable httpd.service
```

### Restart Apache Service

Restart the Apache service when changes are made:

```bash
systemctl restart httpd.service
```

---

## 6. Network and Port Verification

### Check Open Ports and Services

- **List all open ports:**

  ```bash
  netstat -nltup
  ```

- **Check if port 80 is open:**

  ```bash
  netstat -nltup | grep 80
  ```

- **Check if Apache is listening:**

  ```bash
  netstat -nltup | grep httpd
  ```

---

## 7. Firewall Configuration

Firewalld is used to dynamically manage firewall rules. You can allow traffic either by using a predefined service or by opening specific ports.

### 7.1 Allow a Service (Recommended)

Firewalld has predefined services that automatically open the required ports. For example:

```bash
firewall-cmd --add-service=http --permanent
firewall-cmd --add-service=https --permanent
firewall-cmd --reload
```

This configuration allows web traffic on ports **80** and **443**.

### 7.2 Allow a Specific Port

If a service is not predefined, you can open a port manually. For example, allow traffic on TCP port **8080**:

```bash
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --reload
```

> **Note:** If your Apache (or Tomcat) server runs on ports like **8080** or **8443**, you must allow these ports explicitly.

### 7.3 Check Allowed Services & Ports

To see the active firewall configuration:

```bash
firewall-cmd --list-all
```

### 7.4 Remove a Service or Port

To remove the defined service or port:

```bash
firewall-cmd --remove-service=http --permanent
firewall-cmd --remove-port=8080/tcp --permanent
firewall-cmd --reload
```

---

## 8. Enable Logging (Optional)

Enable logging for dropped packets:

```bash
firewall-cmd --set-log-denied=all
```

View the firewall logs in real time:

```bash
journalctl -f -u firewalld
```

---

## 9. Apache Process and User Management

### Check Running Processes

Check Apache processes:

```bash
ps -aux | grep httpd
```

To check open files related to Apache:

```bash
lsof | grep httpd
```

### Check Apache User and Group

To verify the Apache user:

```bash
cat /etc/passwd | grep apache
```

> When installing the httpd service, a user and group named `apache` are usually created. This user is used to run the httpd service.

To verify the Apache group:

```bash
cat /etc/group | grep apache
```

---

## 10. Configure Website Files

### Create or Edit Web Files

Navigate to the document root:

```bash
cd /var/www/html
```

> **Note:**  
> The `/var/www/html/` directory is the default document root for Apache on CentOS 9. If no custom files (like `index.html`, `index.php`, etc.) are present, Apache will serve a default welcome page.

#### Disable the Default Welcome Page

To disable the default welcome page, move or rename the file `/etc/httpd/conf.d/welcome.conf`.

#### Create or Edit `index.html`

```bash
vim /var/www/html/index.html
```

### Remove Existing Default File

If needed, remove the existing file:

```bash
rm -f index.html
```

### Set Ownership of Web Files

Set the ownership of the document root to the Apache user:

```bash
chown -Rv apache:apache /var/www/html/
```

---

## 11. Load Custom Site

### Download and Extract Template Files

- **Download the template:**

  ```bash
  wget https://templatemo.com/tm-zip-files-2020/templatemo_571_hexashop.zip
  ```

- **Unzip the template:**

  ```bash
  unzip templatemo_571_hexashop.zip
  ```

- **Remove the zip file:**

  ```bash
  rm -rf templatemo_571_hexashop.zip
  ```

- **Move the extracted folder:**

  ```bash
  mv -v templatemo_571_hexashop site1
  ```

- **Copy site files to the web root:**

  ```bash
  cp -vr /root/site1/* /var/www/html
  ```

---

## 12. Configure Apache to Load CSS Files (If CSS Does Not Load)

Edit the Apache configuration file (`/etc/httpd/conf/httpd.conf`) to include the appropriate MIME type and filter settings. For example, add the following lines around line 311:

```apache
<IfModule mime_module>
    AddType text/html .shtml
    AddType text/css .css
    AddOutputFilter INCLUDES .shtml
</IfModule>
```

Edit the configuration file:

```bash
vim /etc/httpd/conf/httpd.conf
```

Restart Apache to apply changes:

```bash
systemctl restart httpd.service
```

Set correct ownership for your web files:

```bash
chown -Rv apache:apache /var/www/html/*
```

---

## 13. Test Apache Setup

### Test Configuration

Create a test file (e.g., with the text "Test123 ...") in the document root if needed.

### Check if Apache is Running

Use `curl` to verify that Apache is responding:

```bash
curl http://localhost
```

---

