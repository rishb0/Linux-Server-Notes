# DNS-Server

**Caching Nameserver**
	
	A caching nameserver stores DNS query results temporarily to improve performance and reduce the load on authoritative nameservers. This guide covers the setup and configuration of a caching nameserver using BIND (Berkeley Internet Name Domain).

---

**Check Hostname**

**Display Current Hostname**

```
hostname
```

**Update Hostname**

1. Open the hostname configuration file:

   ```
   vim /etc/hostname
   ```

  ```
  ns1.armour.local
   ```

2. Modify the hostname value as needed.

3. Save and exit.

**Reboot the System**

```
reboot
```

**Confirm the Hostname Change**

```
hostname
```

**Check DNS Resolver Configuration**

**View resolv.conf for DNS Settings**

```
cat /etc/resolv.conf
```

```
nameserver 8.8.8.8
nameserver 8.8.4.4
```

---

**Install and Configure BIND**

**Check if BIND is Installed**

```
dpkg -l | grep bind9
```

**Install BIND and Utilities**

```
apt update
apt install bind9 bind9-utils bind9-doc
```

**Verify Installation**

```
dpkg -l | grep bind9
```

**Get Information About the Installed Package**

```
apt show bind9
```

**List Installed Files**

```
dpkg -L bind9
```

**List Configuration Files**

```
dpkg -L bind9 | grep etc
```

**Key Configuration Files**

- `/etc/bind/named.conf`
- `/etc/bind/named.conf.options`
- `/etc/bind/named.conf.local`
- `/etc/bind/named.conf.default-zones`
- `/etc/bind/db.root` (equivalent to named.ca)
- `/var/cache/bind/` (working directory)
  - `/var/cache/bind/named.stats`
  - `/var/cache/bind/managed-keys.bind`
  - `/var/cache/bind/named.run`
  - `/var/cache/bind/slave/`

**1. /etc/bind/named.conf**

- This is the main configuration file for BIND.
- It includes other configuration files and defines high-level settings.
- In Debian, this file often just includes the other configuration files.

**2. /etc/bind/named.conf.options**

- Contains the global options for the BIND server.
- Defines server behavior, security settings, and network configurations.
- Equivalent to the options section in CentOS's `/etc/named.conf`.

**3. /etc/bind/named.conf.local**

- Used for configuring local zones specific to your setup.
- This is where you define your custom forward and reverse zones.

**4. /etc/bind/named.conf.default-zones**

- Similar to CentOS's `/etc/named.rfc1912.zones`.
- Contains definitions for standard zones like localhost and reverse mapping.

**5. /etc/bind/db.root**

- Contains the list of root DNS servers.
- Equivalent to `/var/named/named.ca` in CentOS.

**6. /var/cache/bind**

- This is the working directory for BIND, where zone files and cache data are stored.
- Equivalent to `/var/named` in CentOS.

**View Cache File**

```
cat /etc/bind/db.root
```

**Manage BIND Service**

**Check Service Status**

```
systemctl status bind9
```

**Start the Service**

```
systemctl start bind9
```

**Check Open Ports for BIND**

```
netstat -nltup | grep named
```

```
netstat -nltup | grep 53
```

**Test Using dig**

**Test Localhost Resolution**

```
dig google.com @127.0.0.1
```

**Configure Network Settings**

**Debian – Modify Network Interface Configuration**

Edit the network configuration file:

```
vim /etc/network/interfaces
```

**Example:**

```
# The primary network interface
auto enp0s3
iface enp0s3 inet static
    address 192.168.1.37
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 127.0.0.1
```

For newer Debian versions using Netplan:

```
vim /etc/netplan/01-netcfg.yaml
```

**Example:**

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: no
      addresses: [192.168.1.37/24]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [127.0.0.1]
```

Apply the configuration:

```
netplan apply
```

**Configure BIND (named.conf)**

**Example named.conf.options File**

```
vim /etc/bind/named.conf.options
```

```
options {
    directory "/var/cache/bind";
    
    listen-on port 53 { 127.0.0.1; 192.168.1.50; };
    listen-on-v6 port 53 { ::1; };
    
    dump-file "/var/cache/bind/cache_dump.db";
    statistics-file "/var/cache/bind/named_stats.txt";
    memstatistics-file "/var/cache/bind/named_mem_stats.txt";
    
    allow-query { localhost; 192.168.1.0/24; };
    
    recursion yes;
    
    dnssec-validation auto;
    
    auth-nxdomain no;    # conform to RFC1035
    
    // Recommended options for improved security
    version none;
    allow-transfer { none; };
};
```

**Configure Firewall Rules (UFW)**

Allow DNS Traffic (TCP and UDP Port 53):

```
ufw allow 53/tcp
```

```
ufw allow 53/udp
```

This will open both TCP and UDP port 53 for DNS queries.

**Define ACL in named.conf.options**

```
vim /etc/bind/named.conf.options
```

**Example:**

```
acl ns_ip_add {
    127.0.0.1;
    192.168.1.34;
    192.168.1.35;
    192.168.1.36;
    192.168.1.37;
};

acl mynetwork {
    127.0.0.1;
    192.168.1.0/24;
};

options {
    directory "/var/cache/bind";
    
    listen-on port 53 { ns_ip_add; };
    listen-on-v6 port 53 { ::1; };
    
    dump-file "/var/cache/bind/cache_dump.db";
    statistics-file "/var/cache/bind/named_stats.txt";
    memstatistics-file "/var/cache/bind/named_mem_stats.txt";
    
    allow-query { mynetwork; };
    
    recursion yes;
    
    dnssec-validation auto;
    
    auth-nxdomain no;    # conform to RFC1035
    
    // Recommended options for improved security
    version none;
    allow-transfer { none; };
};
```

**Restart Services**

**Restart BIND Service**

```
systemctl restart bind9
```

**Restart Firewall**

```
systemctl restart ufw
```

---

**Test Setup**

**Use dig to Test DNS Resolution**

```
dig google.com @192.168.1.50
```

---

Now set dns ip in client as of the server and test resolving names by nslookup of visiting sites etc