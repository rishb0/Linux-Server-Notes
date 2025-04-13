# Reverse-Zone

## Reverse Zone Configuration

# Edit DNS Zones Configuration

Edit the `named.conf.local` file to define the reverse zone:

```bash
vim /etc/bind/named.conf.local
```

Example content:

```
// Define forward zone for armour.local
zone "armour.local" {
    type master;
    file "/var/cache/bind/db.armour.local";
    allow-update { none; };
};

// Define reverse zone for 192.168.1.0/24
zone "1.168.192.in-addr.arpa" {
    type master;
    file "/var/cache/bind/db.192.168.1";
    allow-update { none; };
};
```

# Create Reverse Zone File

```bash
cd /var/cache/bind/
```

Copy the forward zone file and modify it for reverse mapping:

```bash
cp -v /var/cache/bind/db.armour.local /var/cache/bind/db.192.168.1
```

```bash
vim /var/cache/bind/db.192.168.1
```

Example content:

```
$TTL 1D
@       IN SOA  ns1.armour.local. root.armour.local. (
                                        002 ; serial
                                        1D  ; refresh
                                        1H  ; retry
                                        1W  ; expire
                                        3H ) ; minimum

@       IN      NS      ns1.armour.local.
@       IN      PTR     ns1.armour.local.
31      IN      PTR     ns1.armour.local.
1       IN      PTR     router.armour.local.
201     IN      PTR     emp1.armour.local.
202     IN      PTR     emp2.armour.local.
50      IN      PTR     mail.armour.local.
100     IN      PTR     dev.armour.local.
110     IN      PTR     fileserver.armour.local.
120     IN      PTR     db.armour.local.
130     IN      PTR     test.armour.local.
```

# Set Permissions

Set the correct permissions for the reverse zone file:

```bash
chgrp bind /var/cache/bind/db.192.168.1
```

```bash
chmod 640 /var/cache/bind/db.192.168.1
```

# Restart BIND Service

Restart the `bind9` service:

```bash
systemctl restart bind9
```

# Check Configuration

Check the configuration for syntax errors:

```bash
named-checkconf /etc/bind/named.conf
```

# Verify Reverse Zone

Verify the reverse zone file:

```bash
named-checkzone 1.168.192.in-addr.arpa /var/cache/bind/db.192.168.1
```

Output example:

```
zone 1.168.192.in-addr.arpa/IN: loaded serial 002
OK
```

# Firewall Configuration

Allow DNS traffic through the firewall:

* **UFW:**

```
ufw allow 53/tcp
ufw allow 53/udp
```

# Test Reverse Resolution

Test reverse DNS resolution using `dig`:

```
dig -x 192.168.1.100
dig -x 192.168.1.120
```

Example output:

```
; <<>> DiG 9.16.1-Ubuntu <<>> -x 192.168.1.100
;; ANSWER SECTION:
100.1.168.192.in-addr.arpa. 86400 IN    PTR     dev.armour.local.
```

# Explanation

* `zone "1.168.192.in-addr.arpa" { ... }`

    * This creates a reverse zone for the `192.168.1.x` network.

    * The `file` parameter points to the reverse zone file (`db.192.168.1`).

* **PTR Records:**

    * `PTR` - Maps an IP address to a hostname.

* **Test Commands:**

    * `dig -x 192.168.1.100` - Queries the DNS server for reverse resolution.

    * `named-checkzone` - Checks the validity of the reverse zone file.

This config sets up a reverse zone for `192.168.1.x` with proper PTR records and firewall rules.