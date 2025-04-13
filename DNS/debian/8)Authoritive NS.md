# Master-Nameserver

## 🛠️ Master Nameserver Configuration

**Edit Configuration for Master Nameserver**

First, edit the main configuration files in Debian:

```
vim /etc/bind/named.conf.options
```

```
vim /etc/bind/named.conf.local
```

---

**Sample Configuration for Master Nameserver**

Here's the configuration for setting up a master nameserver on Debian:

First, edit the options file (`/etc/bind/named.conf.options`):

```
options {
    directory "/var/cache/bind";
    
    listen-on port 53 { 127.0.0.1; 192.168.1.50; };
    listen-on-v6 port 53 { ::1; };
    
    dump-file "/var/cache/bind/cache_dump.db";
    statistics-file "/var/cache/bind/named_stats.txt";
    memstatistics-file "/var/cache/bind/named_mem_stats.txt";
    
    allow-query { localhost; 192.168.1.0/24; };
    
    // Disable recursion for authoritative server
    recursion no;
    
    dnssec-validation auto;
    
    /* Path to DNSSEC keys */
    bindkeys-file "/etc/bind/bind.keys";
    
    pid-file "/var/run/named/named.pid";
    
    // Recommended options for improved security
    version none;
    allow-transfer { none; };
};
```

Next, edit the local zones file (`/etc/bind/named.conf.local`):

```
// Forward Zone for example.com
zone "example.com" {
    type master;
    file "/var/cache/bind/db.example.com";
    allow-update { none; };
};

// Reverse Zone for 192.168.1.0/24
zone "1.168.192.in-addr.arpa" {
    type master;
    file "/var/cache/bind/db.192.168.1";
    allow-update { none; };
};
```

NOTE:
In Debian, we define zones in /etc/bind/named.conf.local rather than in the main configuration file. Default zones are already included in /etc/bind/named.conf.default-zones.

✅ **Create Forward Zone File**

Create the forward zone file `/var/cache/bind/db.example.com`:

```
vim /var/cache/bind/db.example.com
```

Example content:

```
$TTL 86400
@   IN  SOA ns1.example.com. admin.example.com. (
            2025031101 ; Serial
            3600       ; Refresh
            1800       ; Retry
            604800     ; Expire
            86400 )    ; Minimum TTL

@   IN  NS  ns1.example.com.
ns1 IN  A   192.168.1.50
www IN  A   192.168.1.100
```

---

✅ **Create Reverse Zone File**

Create the reverse zone file `/var/cache/bind/db.192.168.1`:

```
vim /var/cache/bind/db.192.168.1
```

Example content:

```
$TTL 86400
@   IN  SOA ns1.example.com. admin.example.com. (
            2025031101 ; Serial
            3600       ; Refresh
            1800       ; Retry
            604800     ; Expire
            86400 )    ; Minimum TTL

@   IN  NS  ns1.example.com.
50  IN  PTR ns1.example.com.
100 IN  PTR www.example.com.
```

# Set File Permissions

Ensure that the `bind` user owns the zone files:

```bash
chown bind:bind /var/cache/bind/db.example.com
chown bind:bind /var/cache/bind/db.192.168.1
chmod 640 /var/cache/bind/db.example.com
chmod 640 /var/cache/bind/db.192.168.1
```

# Start and Enable BIND Service

Start and enable the BIND service:

```bash
systemctl start bind9
systemctl enable bind9
```

# Test Configuration

Use `dig` to test the DNS resolution:

```bash
dig @192.168.1.50 example.com
dig @192.168.1.50 www.example.com
dig -x 192.168.1.100
```

# Firewall Configuration

Allow DNS traffic through the firewall:

* **UFW:**

```bash
ufw allow 53/tcp
ufw allow 53/udp
```

# Explanation

* `zone "example.com" { type master; ... }`

    * This sets up a master DNS server for the `example.com` zone.

    * The `file` parameter points to the zone file containing the DNS records.

* `zone "1.168.192.in-addr.arpa" { type master; ... }`

    * This sets up reverse DNS resolution for the IP range `192.168.1.0/24`.

* `dig` commands are used to verify that the DNS server is correctly resolving names and addresses.

This config sets up a master nameserver for authoritative DNS resolution — perfect for managing a local network or hosting your own domain!