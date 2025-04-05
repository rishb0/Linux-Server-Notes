# Master-Nameserver

## 🛠️ Master Nameserver Configuration

**Edit Configuration for Master Nameserver**

Edit the `named.conf` file:

```

vim /etc/named.conf

```

---

**Sample Configuration for Master Nameserver**

Here's an example `named.conf` configuration file for setting up a master nameserver:

```

// named.conf

//

// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS

// server as a caching-only nameserver (as a localhost DNS resolver only).

//

// See /usr/share/doc/bind*/sample/ for example named configuration files.

//

// See the BIND Administrator's Reference Manual (ARM) for details about the

// configuration located in /usr/share/doc/bind-{version}/Bv9ARM.html

options {

    listen-on port 53 { 127.0.0.1; 192.168.1.50; };

    listen-on-v6 port 53 { ::1; };

    directory       "/var/named";

    dump-file       "/var/named/data/cache_dump.db";

    statistics-file "/var/named/data/named_stats.txt";

    memstatistics-file "/var/named/data/named_mem_stats.txt";

    recursing-file  "/var/named/data/named.recursing";

    secroots-file   "/var/named/data/named.secroots";

    allow-query     { localhost; 192.168.1.0/24; };

    // Disable recursion for authoritative server

    recursion no;

    dnssec-enable yes;

    dnssec-validation yes;

    /* Path to ISC DLV key */

    bindkeys-file "/etc/named.root.key";

    managed-keys-directory "/var/named/dynamic";

    pid-file "/run/named/named.pid";

    session-keyfile "/run/named/session.key";

};

logging {

    channel default_debug {

        file "data/named.run";

        severity dynamic;

    };

};

// Root Zone

zone "." IN {

    type hint;

    file "named.ca";

};

// Forward Zone for example.com

zone "example.com" IN {

    type master;

    file "/var/named/example.com.zone";

    allow-update { none; };

};

// Reverse Zone for 192.168.1.0/24

zone "1.168.192.in-addr.arpa" IN {

    type master;

    file "/var/named/1.168.192.zone";

    allow-update { none; };

};

include "/etc/named.rfc1912.zones";

include "/etc/named.root.key";

```

NOTE :

We can define Forward Zone or Reverse zone  in /etc/named.conf or    /etc/named.rfc1912.zones ( default zone file )also .

✅ **Create Forward Zone File**

Create the forward zone file `/var/named/example.com.zone`:

```

vim /var/named/example.com.zone

```

Example content:

```

$TTL 86400

@   IN  SOA ns1.example.com. admin.example.com. (

            2025031101 ; Serial

            3600       ; Refresh

            1800       ; Retry

            604800     ; Expire

            86400 )    ; Minimum TTL

@   IN  NS  ns1.example.com.

ns1 IN  A   192.168.1.50

www IN  A   192.168.1.100

```

---

✅ **Create Reverse Zone File**

Create the reverse zone file `/var/named/1.168.192.zone`:

```

vim /var/named/1.168.192.zone

```

Example content:

```

$TTL 86400

@   IN  SOA ns1.example.com. admin.example.com. (

            2025031101 ; Serial

            3600       ; Refresh

            1800       ; Retry

            604800     ; Expire

            86400 )    ; Minimum TTL

@   IN  NS  ns1.example.com.

50  IN  PTR ns1.example.com.

100 IN  PTR [www.example.com](http://www.example.com).

```

# Set File Permissions

Ensure that the `named` user owns the zone files:

```bash

chown named:named /var/named/example.com.zone

chown named:named /var/named/1.168.192.zone

chmod 640 /var/named/example.com.zone

chmod 640 /var/named/1.168.192.zone

```

# Start and Enable BIND Service

Start and enable the BIND service:

```bash

systemctl start named

systemctl enable named

```

# Test Configuration

Use `dig` to test the DNS resolution:

```bash

dig @192.168.1.50 example.com

dig @192.168.1.50 [www.example.com](http://www.example.com)

dig -x 192.168.1.100

```

# Firewall Configuration

Allow DNS traffic through the firewall:

* **UFW:**

```bash

ufw allow 53/tcp

ufw allow 53/udp

```

* **Firewalld:**

```bash

firewall-cmd --add-port=53/tcp --permanent

firewall-cmd --add-port=53/udp --permanent

firewall-cmd --reload

```

# Explanation

* `zone "example.com" IN { type master; ... }`

    * This sets up a master DNS server for the `example.com` zone.

    * The `file` parameter points to the zone file containing the DNS records.

* `zone "1.168.192.in-addr.arpa" IN { type master; ... }`

    * This sets up reverse DNS resolution for the IP range `192.168.1.0/24`.

* `dig` commands are used to verify that the DNS server is correctly resolving names and addresses.

This config sets up a master nameserver for authoritative DNS resolution — perfect for managing a local network or hosting your own domain!