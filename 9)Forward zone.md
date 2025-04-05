# Forward-Zone

## Forward Zone Configuration

# Set Hostname

Set the hostname for the DNS server:

```bash

vim /etc/hostname

```

Example content:

```

ns1.armour.local

```

# Edit Hosts File

Add the hostname mapping in `/etc/hosts`:

```bash

vim /etc/hosts

```

Example content:

```

127.0.0.1       localhost ns1 ns1.armour.local

::1             localhost localhost6 ns1 ns1.armour.local

192.168.1.31    ns1 ns1.armour.local

```

# Edit DNS Zones Configuration

Edit the `named.rfc1912.zones` file to define the forward zone:

```bash

vim /etc/named.rfc1912.zones

```

Example content:

```

// named.rfc1912.zones:

//

// Provided by Red Hat caching-nameserver package

//

// ISC BIND named zone configuration for zones recommended by

// RFC 1912 section 4.1 : localhost TLDs and address zones

// and [https://tools.ietf.org/html/rfc6303](https://tools.ietf.org/html/rfc6303)

//

// (c)2007 R W Franks

//

zone "localhost.localdomain" IN {

type master;

file "named.localhost";

allow-update { none; };

};

```

zone "localhost" IN {

type master;

file "named.localhost";

allow-update { none; };

};

zone "1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.ip6.arpa" IN {

type master;

file "named.loopback";

allow-update { none; };

};

zone "1.0.0.127.in-addr.arpa" IN {

type master;

file "named.loopback";

allow-update { none; };

};

zone "0.in-addr.arpa" IN {

type master;

file "named.empty";

allow-update { none; };

};

zone "armour.local" IN {

type master;

file "forward.armour.local";

allow-update { none; };

};

```

Note :

In configure file , the zone file name we give, can be full path of can also be only file name as it by default know to check in /var/named/<here>

# Create Forward Zone File

Copy the example zone file and modify it:

```



```
cd /var/named/
```

cp -v named.localhost forward.armour.local

```

- This is the sample file which we copy and use it to configure our zone file ,

```

vim /var/named/forward.armour.local

```

Example content:

```

$TTL 1D

@       IN SOA  ns1.armour.local. root.armour.local. (

							202250313 ; serial

							3600       ; refresh

							1800       ; retry

							604800     ; expire

							86400 )    ; minimum

@       IN      NS      ns1.armour.local.

ns1     IN      A       192.168.1.34

armour.local.  IN      A       192.168.1.34

www     IN      CNAME   armour.local.

router  IN      A       192.168.1.1

emp1    IN      A       192.168.1.200

emp2    IN      A       192.168.1.201

; Mail exchange

@       IN      MX      10 mail.armour.local.

mail    IN      A       192.168.1.50

; Text record for SPF

@       IN      TXT     "v=spf1 mx a ~all"

; Additional Services

ftp         IN      CNAME   armour.local.

dev         IN      A       192.168.1.100

fileserver  IN      A       192.168.1.110

db          IN      A       192.168.1.120

test        IN      A       192.168.1.130

vpn         IN      A       192.168.1.140

git         IN      A       192.168.1.150

webapp      IN      A       192.168.1.160

logs        IN      A       192.168.1.170

```

Note:

In this file @ represent the hostname itself so replace it with the FQDN of the host. Means like in windows here  @ = same as the parent domain name

# Set Permissions

Set the correct permissions for the zone file:

```

chgrp named /var/named/forward.armour.local

```

```

chmod 640 /var/named/forward.armour.local

```

# Restart BIND Service

Restart the `named` service:

```

systemctl restart named

```

# Check Configuration

Check the configuration for syntax errors:

```

named-checkconf

```

```

named-checkconf /etc/named.conf

```

```

named-checkconf /etc/named.rfc1912.zones

```

```

named-checkconf /etc/named.root.key

```

# Verify Forward Zone

Verify the forward zone file:

```

named-checkzone armour.local /var/named/forward.armour.local

```

Output example:

```

zone armour.local/IN: loaded serial 202250313

OK

```

# Firewall Configuration

Allow DNS traffic through the firewall:

* **UFW:**

```

ufw allow 53/tcp

ufw allow 53/udp

```

* **Firewalld:**

```

firewall-cmd --add-port=53/tcp --permanent

```

```

firewall-cmd --add-port=53/udp --permanent

```

```

firewall-cmd --reload

```

# Test Resolution

Test forward DNS resolution with `dig`:

```bash

dig @192.168.1.34 armour.local

dig @192.168.1.34 emp1.armour.local

dig @192.168.1.34 [ftp.armour.local](ftp://ftp.armour.local)

```

**Example Output:**

```

; <<>> DiG 9.16.23-RH <<>> @192.168.1.34 [ftp.armour.local](ftp://ftp.armour.local)

;; global options: +cmd

;; Got answer:

;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 20826

;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:

; EDNS: version: 0, flags:; udp: 1232

; COOKIE: afda7df953608d5d0100000067d283868cdf9b5f7dbe657f (good)

;; QUESTION SECTION:

;[ftp.armour.local](ftp://ftp.armour.local).            IN      A

;; ANSWER SECTION:

[ftp.armour.local](ftp://ftp.armour.local).      86400  IN      CNAME   armour.local.

armour.local.      86400  IN      A       192.168.1.34

;; Query time: 0 msec

;; SERVER: 192.168.1.34#53(192.168.1.34)

;; WHEN: Thu Mar 13 12:34:38 IST 2025

;; MSG SIZE  rcvd: 103

# Explanation

* `zone "armour.local" IN { ... }`

* This creates a forward zone for the `armour.local` domain.

* The `file` parameter points to the forward zone file (`forward.armour.local`).

* **Zone Records:**

* `SOA` — Start of Authority record.

* `NS` — Nameserver record.

* `A` — IPv4 address record.

* `CNAME` — Alias record.

* `MX` — Mail exchange record.

* `TXT` — Text record for SPF configuration.

* **Test Commands:**

* `dig @192.168.1.31` — Queries the DNS server directly.

* `named-checkzone` — Checks the validity of the zone file.

```

This config sets up a **forward zone** for `armour.local` with various resource records (A, CNAME, MX, TXT) and proper firewall rules.