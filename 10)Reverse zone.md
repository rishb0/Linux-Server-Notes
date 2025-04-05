# Reverse-Zone

## Reverse Zone Configuration

# Edit DNS Zones Configuration

Edit the `named.rfc1912.zones` file to define the reverse zone:

`vim /etc/named.rfc1912.zones`

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

# Create Reverse Zone File

`cd /var/named/`

Copy the forward zone file and modify it for reverse mapping:

`cp -v /var/named/forward.armour.local /var/named/reverse.armour.local`

`vim /var/named/reverse.armour.local`

Example content:

```

$TTL 1D

@       IN SOA  ns1.armour.local. root.armour.local. (

                                        002 ; serial

                                        1D  ; refresh

                                        1H  ; retry

                                        1W  ; expire

                                        3H ) ; minimum

@       IN      NS      ns1.armour.local.

@       IN      PTR     ns1.armour.local.

31      IN      PTR     ns1.armour.local.

1       IN      PTR     router.armour.local.

201     IN      PTR     emp1.armour.local.

202     IN      PTR     emp2.armour.local.

50.1.168.192.in-addr.arpa.     IN      PTR     mail.armour.local.

100.1.168.192.in-addr.arpa.    IN      PTR     dev.armour.local.

110.1.168.192.in-addr.arpa.    IN      PTR     fileserver.armour.local.

120.1.168.192.in-addr.arpa.    IN      PTR     db.armour.local.

130.1.168.192.in-addr.arpa.    IN      PTR     test.armour.local.

```

# Set Permissions

Set the correct permissions for the reverse zone file:

`chgrp named /var/named/reverse.armour.local`

`chmod 640 /var/named/reverse.armour.local`

# Restart BIND Service

Restart the `named` service:

`systemctl restart named`

# Check Configuration

Check the configuration for syntax errors:

`named-checkconf /etc/named.conf`

# Verify Reverse Zone

Verify the reverse zone file:

`named-checkzone 1.168.192.in-addr.arpa /var/named/reverse.armour.local`

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

* **Firewalld:**

```

firewall-cmd --add-port=53/tcp --permanent

firewall-cmd --add-port=53/udp --permanent

firewall-cmd --reload

```

# Test Reverse Resolution

Test reverse DNS resolution using `dig`:

```

dig -x 192.168.1.100

dig -x 192.168.1.120

```

Example output:

```

; <<>> DiG 9.11.13-RedHat-9.11.13-5.el7 <<>> -x 192.168.1.100

;; ANSWER SECTION:

100.1.168.192.in-addr.arpa. 86400 IN    PTR     dev.armour.local.

```

# Explanation

* `zone "1.168.192.in-addr.arpa" IN { ... }`

    * This creates a reverse zone for the `192.168.1.x` network.

    * The `file` parameter points to the reverse zone file (`reverse.armour.local`).

* **PTR Records:**

    * `PTR` - Maps an IP address to a hostname.

* **Test Commands:**

    * `dig -x 192.168.1.100` - Queries the DNS server for reverse resolution.

    * `named-checkzone` - Checks the validity of the reverse zone file.

This config sets up a reverse zone for `192.168.1.x` with proper PTR records and firewall rules.