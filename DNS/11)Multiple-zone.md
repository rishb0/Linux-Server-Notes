# Multiple-Zones
## Multiple Zone Configuration

### Setup for `infosec.local`
1. **Edit DNS Zones Configuration**  
   Edit the `named.rfc1912.zones` file to add the `infosec.local` zone:
   ```bash
   vim /etc/named/rfc1912.zones
   ```
   Example content:
   ```bash
   zone "infosec.local" IN {
       type master;
       file "forward.infosec.local";
       allow-update { none; };
   };
   ```

2. **Create Forward Zone File**  
   Copy the existing forward zone file:
   ```bash
   cp -v /var/named/forward.armour.local /var/named/forward.infosec.local
   ```
   Edit the new zone file:
   ```bash
   vim /var/named/forward.infosec.local
   ```
   Example content:
 ```bash
   $TTL 1D
   $ORIGIN infosec.local.
   @ IN SOA ns1.armour.local. root.armour.local. (
       002 ; serial
       1D  ; refresh
       1H  ; retry
       1W  ; expire
       3H ) ; minimum
   @ IN NS ns1.armour.local.
   infosec.local. IN A 192.168.1.200
   www IN CNAME infosec.local.

   ; Mail exchange
   @ IN MX 10 mail.armour.local.

   ; Text record for SPF
   @ IN TXT "v=spf1 mx a ~all"

   ; Additional Services
   ftp        IN A 192.168.1.112
   dev        IN A 192.168.1.111
   fileserver IN A 192.168.1.110
   db         IN A 192.168.1.120
   test       IN A 192.168.1.130
   vpn        IN A 192.168.1.140
   git        IN A 192.168.1.150
   webapp     IN A 192.168.1.160
   logs       IN A 192.168.1.170
 ```
# 3. Set Permissions

Set appropriate permissions for the zone file:

    chgrp named /var/named/forward.infosec.local

    chmod 640 /var/named/forward.infosec.local

# 4. Validate Configuration

Check the configuration:

    named-checkconf /etc/named.conf

    named-checkconf /etc/named.rfc1912.zones

    named-checkzone infosec.local /var/named/forward.infosec.local

Example output:

    zone infosec.local/IN: loaded serial 002
    OK

# 5. Restart BIND

Restart the named service:

    systemctl restart named.service

# 6. Test the Setup

Test forward DNS resolution:

    dig infosec.local

    dig mx infosec.local

    dig txt infosec.local

Example output:

    ; <<>> DiG 9.11.13-RedHat-9.11.13-5.el7 <<>> infosec.local
    ;; ANSWER SECTION:
    infosec.local.    86400   IN  A   192.168.1.200

# ✓ Setup for ai.local

# 1. Edit DNS Zones Configuration

Edit named.rfc1912.zones to add the ai.local zone:

    vim /etc/named.rfc1912.zones
    
Example content:

    zone "ai.local" IN {
        type master;
        file "forward.ai.local";
        allow-update { none; };
    };

# 2. Create Forward Zone File

Copy the infosec.local zone file:

    cp -v /var/named/forward.infosec.local /var/named/forward.ai.local

Edit the new zone file:

    vim /var/named/forward.ai.local

Example content:

    $TTL 1D
    $ORIGIN ai.local.
    @   IN SOA  ns1.armour.local. root.armour.local. (
                    002 ; serial
                    1D  ; refresh
                    1H  ; retry
                    1W  ; expire
                    3H  ) ; minimum
    @           IN NS   ns1.armour.local.
    ai.local.   IN A    192.168.1.201
    www         IN CNAME ai.local.

    ; Mail exchange
    @           IN MX 10 mail.armour.local.

    ; Text record for SPF
    @           IN TXT "v=spf1 mx a -all"

    ; Additional Services
    ftp         IN A    192.168.1.112
    dev         IN A    192.168.1.111
    fileserver  IN A    192.168.1.110
    db          IN A    192.168.1.120
    webapp      IN A    192.168.1.160
    logs        IN A    192.168.1.170

# 3. Set Permissions

Set permissions for the zone file:

    chgrp named /var/named/forward.ai.local
 
	chmod 640 /var/named/forward.ai.local
# 4. Validate Configuration

Check the configuration:

    named-checkconf /etc/named.conf
    named-checkconf /etc/named.rfc1912.zones
    named-checkzone ai.local /var/named/forward.ai.local

Example output:

    zone ai.local/IN: loaded serial 002
    OK

# 5. Restart BIND

Restart the named service:

    systemctl restart named

# 6. Test the Setup

Test forward DNS resolution:

    dig ai.local

	dig mx ai.local
    
    dig txt ai.local

Example output:

    ; <<>> DiG 9.11.13-RedHat-9.11.13-5.el7 <<>> ai.local
    ;; ANSWER SECTION:
    ai.local.    86400   IN  A   192.168.1.201

# ✓ Firewall Configuration

Allow DNS traffic through the firewall:

* UFW:

    ufw allow 53/tcp
    ufw allow 53/udp

* Firewalld:

    firewall-cmd --add-port=53/tcp --permanent
    firewall-cmd --add-port=53/udp --permanent
    firewall-cmd --reload

# ✓ Explanation

1. Zone Setup:
   * zone "infosec.local" IN - Defines the infosec.local zone.
   * zone "ai.local" IN - Defines the ai.local zone.

2. Record Types:
   * A - Maps a hostname to an IPv4 address.
   * CNAME - Alias for another domain name.
   * MX - Mail server record.
   * TXT - Text record (e.g., for SPF).

3. Testing:
   * dig - Used for querying DNS records.

# ✓ Summary

✓ Created multiple zones: infosec.local and ai.local
✓ Configured DNS records (A, CNAME, MX, TXT)
✓ Validated configuration with named-checkzone
✓ Tested using dig

This setup ensures that both infosec.local and ai.local zones are properly configured and operational ✓

	The $ORIGIN directive in a DNS zone file sets the base domain name for any relative records defined in the file.

# Explanation:

    $ORIGIN ai.local.

* The $ORIGIN directive defines the default domain name for the zone file.
* Any DNS records that do not end with a period (.) are automatically appended with the $ORIGIN value.

# Example:

Given this snippet from the forward.ai.local zone file:

    $ORIGIN ai.local.
    @   IN SOA ns1.armour.local. root.armour.local. (
                002 ; serial
                1D  ; refresh
                1H  ; retry
                1W  ; expire
                3H )  ; minimum
    @           IN NS   ns1.armour.local.
    ai.local.   IN A    192.168.1.201
    www         IN CNAME ai.local.
    ftp         IN A    192.168.1.112

1. @ refers to the current $ORIGIN, which is ai.local.
2. www IN CNAME ai.local. becomes www.ai.local.
3. ftp IN A 192.168.1.112 becomes ftp.ai.local.

If the $ORIGIN directive wasn't set, you'd need to explicitly define the full domain names for every record, like:

    www.ai.local.    IN CNAME    ai.local.
    ftp.ai.local.    IN A        192.168.1.112

# ✓ Purpose of $ORIGIN:
* Helps avoid repetition of the full domain name.
* Simplifies the zone file, making it easier to manage.
* Ensures consistency and reduces errors.
