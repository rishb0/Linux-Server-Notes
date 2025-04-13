# Master-and-Slave-DNS-Server
## Master and Slave DNS Server

This guide covers the configuration of a Master DNS Server and a Slave DNS Server using BIND (Berkeley Internet Name Domain).

# Server Details

| Role | IP Address | Description |
|------|------------|-------------|
| Master DNS | 192.168.1.34 | Primary DNS server |
| Slave DNS | 192.168.1.3 | Secondary DNS server |
| Domain Name | armour.local | Example domain |

# Master DNS Configuration

## Step 1: Set the Hostname

Set the hostname for the Master DNS server:

    hostnamectl set-hostname ns1.armour.local

## Step 2: Install BIND

Install the BIND package:

    apt update
    apt install bind9 bind9-utils -y

## Step 3: Configure the DNS Zones

Edit the zone configuration file:

    vim /etc/bind/named.conf.local

Example configuration:

    zone "armour.local" {
        type master;
        file "/var/cache/bind/db.armour.local";
        allow-update { none; };
        allow-transfer { 192.168.1.35; };
    };

    zone "infosec.local" {
        type master;
        file "/var/cache/bind/db.infosec.local";
        allow-update { none; };
        allow-transfer { 192.168.1.35; };
    };

    zone "ai.local" {
        type master;
        file "/var/cache/bind/db.ai.local";
        allow-update { none; };
        allow-transfer { 192.168.1.35; };
    };

# Step 4: Create the Zone Files

Create the forward lookup zone files:

## Forward Zone for armour.local

    vim /var/cache/bind/db.armour.local

Example content:

    $TTL 1D
    @   IN SOA  ns1.armour.local. root.armour.local. (
                    20250313    ; serial
                    3600        ; refresh
                    1800        ; retry
                    604800      ; expire
                    86400 )     ; minimum
    @           IN NS   ns1.armour.local.
    @           IN NS   ns2.armour.local.
    ns1         IN A    192.168.1.34
    ns2         IN A    192.168.1.38
    armour.local.   IN A    192.168.1.34
    www         IN CNAME armour.local.
    router      IN A    192.168.1.1
    emp1        IN A    192.168.1.200
    emp2        IN A    192.168.1.201

    ; Mail exchange
    @           IN MX 10 mail.armour.local.
    mail        IN A    192.168.1.50

    ; Text record for SPF
    @           IN TXT "v=spf1 mx a -all"

    ; Additional Services
    ftp         IN CNAME armour.local.
    dev         IN A    192.168.1.100
    fileserver  IN A    192.168.1.110
    db          IN A    192.168.1.120
    test        IN A    192.168.1.130
    vpn         IN A    192.168.1.140
    git         IN A    192.168.1.150
    webapp      IN A    192.168.1.160
    logs        IN A    192.168.1.170

## Forward Zone for infosec.local

    vim /var/cache/bind/db.infosec.local

Example content:

    $TTL 1D
    $ORIGIN infosec.local.
    @   IN SOA  ns1.armour.local. root.armour.local. (
                    20250324    ; serial
                    3600        ; refresh
                    1800        ; retry
                    604800      ; expire
                    86400 )     ; minimum
    @           IN NS   ns1.armour.local.
    @           IN NS   ns2.armour.local.
    infosec.local.   IN A    192.168.1.35
    www         IN CNAME infosec.local.
    router      IN A    192.168.1.1
    emp1        IN A    192.168.1.200
    emp2        IN A    192.168.1.201

    ; Mail exchange
    @           IN MX 10 mail.armour.local.

    ; Text record for SPF
    @           IN TXT "v=spf1 mx a -all"

    ; Additional Services
    ftp         IN CNAME infosec.local.
    dev         IN A    192.168.1.100
    fileserver  IN A    192.168.1.110
    db          IN A    192.168.1.120
    test        IN A    192.168.1.130
    vpn         IN A    192.168.1.140
    git         IN A    192.168.1.150
    webapp      IN A    192.168.1.160
    logs        IN A    192.168.1.170
## Forward Zone for ai.local

    vim /var/cache/bind/db.ai.local

Example content:

    $TTL 1D
    $ORIGIN ai.local.
    @   IN SOA  ns1.armour.local. root.armour.local. (
                    20250325    ; serial
                    3600        ; refresh
                    1800        ; retry
                    604800      ; expire
                    86400 )     ; minimum
    @           IN NS   ns1.armour.local.
    @           IN NS   ns2.armour.local.
    ai.local.   IN A    192.168.1.35
    www         IN CNAME ai.local.
    router      IN A    192.168.1.1
    emp1        IN A    192.168.1.200
    emp2        IN A    192.168.1.201

    ; Mail exchange
    @           IN MX 10 mail.armour.local.

    ; Text record for SPF
    @           IN TXT "v=spf1 mx a -all"

    ; Additional Services
    ftp         IN CNAME ai.local.
    dev         IN A    192.168.1.100
    fileserver  IN A    192.168.1.110
    db          IN A    192.168.1.120
    test        IN A    192.168.1.130
    vpn         IN A    192.168.1.140
    git         IN A    192.168.1.150
    webapp      IN A    192.168.1.160
    logs        IN A    192.168.1.170
# Step 5: Configure Firewall Rules (UFW)

Allow DNS Traffic (TCP and UDP Port 53):

    ufw allow 53/tcp
    ufw allow 53/udp

# Step 6: Validate Configuration

Check the configuration files:

    named-checkconf
    named-checkconf /etc/bind/named.conf
    named-checkconf /etc/bind/named.conf.local
    named-checkzone armour.local /var/cache/bind/db.armour.local
    named-checkzone infosec.local /var/cache/bind/db.infosec.local

# Step 7: Start and Enable BIND

Start and enable the BIND service:

    systemctl restart bind9
    systemctl enable bind9

# Slave DNS Configuration

## Step 1: Set the Hostname

Set the hostname for the Slave DNS server:

    hostnamectl set-hostname ns2.armour.local

## Step 2: Install BIND

Install the BIND package:

    apt update
    apt install bind9 bind9-utils -y

## Step 3: Configure /etc/hosts

Edit the /etc/hosts file:

    vim /etc/hosts

Example content:

    127.0.0.1   localhost ns2 ns2.armour.local
    ::1         localhost localhost6 ns2 ns2.armour.local
    192.168.1.35 ns2 ns2.armour.local

# Step 4: Configure the Zones on the Slave

Edit the zone configuration file:

    vim /etc/bind/named.conf.local

Example content:

    zone "armour.local" {
        type slave;
        masters { 192.168.1.34; };
        file "slaves/db.armour.local";
    };

    zone "infosec.local" {
        type slave;
        masters { 192.168.1.34; };
        file "slaves/db.infosec.local";
    };
  
    zone "ai.local" {
        type slave;
        masters { 192.168.1.34; };
        file "slaves/db.ai.local";
    };

# Step 5: Configure Firewall Rules (UFW)

Allow DNS Traffic (TCP and UDP Port 53):

    ufw allow 53/tcp
    ufw allow 53/udp

# Step 6: Start and Enable BIND

Start and enable the BIND service:

    systemctl restart bind9
    systemctl enable bind9

# Step 7: Validate Configuration

Check the configuration files:

    named-checkconf
    named-checkconf /etc/bind/named.conf
    named-checkconf /etc/bind/named.conf.local

# Step 8: Test DNS Resolution

Test DNS resolution from the slave:

    dig armour.local @192.168.1.38

    dig infosec.local @192.168.1.38

    dig ai.local @192.168.1.38

These commands will test DNS resolution by querying the slave DNS server (located at 192.168.1.38) for each of the configured domains. This verifies that the slave server has properly received and can serve the zone information from the master server.
# Troubleshooting

## Check DNS Service Status

    systemctl status bind9

## Check Listening Ports

    netstat -nltup | grep named

## Test External DNS Resolution

    dig google.com
    dig google.com @192.168.1.35

# Conclusion

This completes the setup of a Master and Slave DNS server using BIND. Ensure that DNS resolution is working correctly and that the firewall rules are properly configured.