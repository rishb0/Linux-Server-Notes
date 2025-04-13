# DNS Notes for CentOS 9

> _This document covers DNS server types, client tools, record types, BIND configuration (caching, authoritative, master/slave, multiple zones), DDNS configuration (both client- and DHCP server–initiated), firewall rules, and troubleshooting steps. All examples are tailored for CentOS 9._

---

## Table of Contents

1. [Introduction](#introduction)
2. [DNS Server Types](#dns-server-types)
   - [Non-Authoritative (Recursive) Nameservers](#non-authoritative-recursive-nameservers)
   - [Caching Nameservers](#caching-nameservers)
   - [Forwarders Nameservers](#forwarders-nameservers)
   - [Authoritative Nameservers](#authoritative-nameservers)
     - [Primary/Master Nameserver](#primarymaster-nameserver)
     - [Secondary/Slave Nameserver](#secondaryslave-nameserver)
3. [DNS Client Tools](#dns-client-tools)
   - [Common Tools: dig, nslookup, host, mdig](#common-dns-client-tools)
4. [DNS Records Overview](#dns-records-overview)
   - [A and AAAA Records](#a-and-aaaa-records)
   - [CNAME Records](#cname-records)
   - [NS Records](#ns-records)
   - [PTR Records (Reverse DNS)](#ptr-records)
   - [SOA Records](#soa-records)
   - [HINFO Records](#hinfo-records)
   - [MX Records](#mx-records)
   - [TXT Records](#txt-records)
5. [Using dig – Examples & Output Explained](#using-dig)
6. [Caching Nameserver Setup and Configuration](#caching-nameserver-configuration)
   - [Hostname and /etc/resolv.conf](#hostname-and-resolvconf)
   - [BIND Installation and Verification](#bind-installation-and-verification)
7. [Detailed BIND Configuration Files and Directories](#bind-configuration)
   - [Key Files and Directories](#key-files-and-directories)
   - [Sample /etc/named.conf Explanation](#sample-namedconf-explanation)
8. [Authoritative DNS Server Configuration (Master Nameserver)](#master-nameserver-configuration)
   - [Example named.conf for Master](#master-namedconf)
   - [Forward Zone File Creation](#forward-zone-file-creation)
   - [Reverse Zone File Creation](#reverse-zone-file-creation)
9. [Multiple-Zones Configuration](#multiple-zones-configuration)
   - [Setting Up infosec.local](#setup-infoseclocal)
   - [Setting Up ai.local](#setup-ailocal)
   - [$ORIGIN Directive Explanation](#origin-directive-explanation)
10. [Master & Slave DNS Server Setup](#master-slave-setup)
    - [Master DNS Setup](#master-dns-setup)
    - [Slave DNS Setup](#slave-dns-setup)
11. [Firewall Configuration for DNS](#firewall-configuration)
12. [Dynamic DNS (DDNS) Configuration](#ddns-configuration)
    - [Client-Initiated DDNS Updates](#client-initiated-ddns)
    - [DHCP Server-Initiated DDNS Updates](#dhcp-server-initiated-ddns)
    - [TSIG Key Generation and Explanation](#tsig-key-generation)
13. [DNS Troubleshooting and Testing](#dns-troubleshooting)
14. [Conclusion and Summary](#conclusion)

---

## 1. Introduction

DNS (Domain Name System) translates human-friendly domain names (like example.com) to IP addresses required to locate computer services. This document provides a full explanation—from server types and record types to advanced configuration setups like DDNS (Dynamic DNS) and master-slave zone transfers—tailored for CentOS 9.

---

## 2. DNS Server Types

DNS servers are classified based on their function and the type of DNS data they serve.

### Non-Authoritative (Recursive) Nameservers

- They resolve DNS queries by performing a full lookup, contacting multiple servers until an authoritative answer is found.
- They do not hold original zone data.

### Caching Nameservers

- **Description:**  
  These store DNS query results temporarily to speed up queries and reduce load on authoritative servers.
- **Example command to test using a caching nameserver:**
  ```bash
  dig example.com @8.8.8.8
  ```

### Forwarders Nameservers

- **Description:**  
  A forwarder receives DNS queries and passes them to another DNS server (often a caching or authoritative server) for resolution.
- **Usage Example:**  
  Add a nameserver entry in `/etc/resolv.conf`:
  ```bash
  echo "nameserver 8.8.8.8" >> /etc/resolv.conf
  ```

### Authoritative Nameservers

Authoritative servers provide definitive answers for the domains they manage. They contain the actual zone records.

#### Primary/Master Nameserver

- **Definition:**  
  The master nameserver holds the original copies of zone data and is responsible for zone updates and administration.
- **Example Configuration:**
  ```named
  zone "example.com" {
      type master;
      file "/etc/bind/db.example.com";
  };
  ```

#### Secondary/Slave Nameserver

- **Definition:**  
  The slave nameserver obtains its data from the master via zone transfers (AXFR) and acts as backup.
- **Example Configuration:**
  ```named
  zone "example.com" {
      type slave;
      file "/etc/bind/db.example.com";
      masters { 192.168.1.1; };
  };
  ```

---

## 3. DNS Client Tools

DNS client tools help query and troubleshoot DNS. The most commonly-used commands include:

### Common DNS Client Tools

- **dig:**  
  A versatile DNS lookup tool.
- **nslookup:**  
  A legacy DNS lookup tool.
- **host:**  
  A simple utility for DNS lookups.
- **mdig:**  
  A tool for performing parallel DNS queries.

Other commands include those to check package installation, list configuration files (`rpm -qc bind-utils` on RPM systems, or `dpkg -L` on Debian).

---

## 4. DNS Records Overview

DNS records define how domain names map to various resources. Here are the main types:

### A and AAAA Records

- **A Record:**  
  Maps a domain to an IPv4 address.  
  _Example:_  
  ```plain
  example.com. IN A 192.168.1.1
  ```
- **AAAA Record:**  
  Maps a domain to an IPv6 address.  
  _Example:_  
  ```plain
  example.com. IN AAAA 2001:db8::1
  ```

### CNAME Records

- **Description:**  
  An alias record that points one domain to another; lookups continue with the target.
- _Example:_  
  ```plain
  www.example.com. IN CNAME example.com.
  ```

### NS Records

- **Description:**  
  Specify the authoritative nameserver for a domain.
- _Example:_  
  ```plain
  example.com. IN NS ns1.example.com.
  ```

### PTR Records

- **Description:**  
  Used in reverse DNS lookups to map an IP address back to a hostname.
- _Example:_  
  ```plain
  1.1.168.192.in-addr.arpa. IN PTR example.com.
  ```

### SOA Records

- **Description:**  
  The Start of Authority record contains administrative information about the zone (primary nameserver, email address, refresh, retry, expire, and minimum TTL).
- _Example:_  
  ```plain
  example.com. IN SOA ns1.example.com. admin.example.com. (
                    2025031101 ; serial
                    3600       ; refresh
                    1800       ; retry
                    604800     ; expire
                    86400      ; minimum TTL
  )
  ```

### HINFO Records

- **Description:**  
  Provide host hardware and OS information (rarely used nowadays due to privacy and security concerns).
- _Example:_  
  ```plain
  example.com. IN HINFO "Intel i7" "Linux"
  ```

### MX Records

- **Description:**  
  Define the mail servers responsible for receiving email for a domain. Lower priority values are preferred.
- _Example:_  
  ```plain
  example.com. IN MX 10 mail.example.com.
  ```

### TXT Records

- **Description:**  
  Hold arbitrary text data, commonly used for SPF, DKIM, or other verification purposes.
- _Example:_  
  ```plain
  example.com. IN TXT "v=spf1 include:_spf.google.com ~all"
  ```

---

## 5. Using dig – Examples & Output Explained

The `dig` command is a powerful tool for querying DNS and diagnosing issues.

### Basic Commands

- **Simple Lookup:**
  ```bash
  dig example.com
  ```
- **Lookup with a Specific DNS Server:**
  ```bash
  dig example.com @8.8.8.8
  ```
- **Request Specific Record Types:**
  ```bash
  dig example.com A
  dig example.com AAAA
  dig example.com NS
  dig example.com MX
  dig example.com SOA
  dig example.com TXT
  ```
- **Short Answer Format:**
  ```bash
  dig example.com +short
  ```
- **Reverse DNS Lookup:**
  ```bash
  dig -x 8.8.8.8 +short
  ```

When you run `dig example.com @8.8.8.8`, the output includes sections such as the QUESTION, ANSWER, and authority sections. The ANSWER section provides the IP addresses corresponding to the domain.

---

## 6. Caching Nameserver Setup and Configuration

A caching nameserver temporarily stores DNS responses to reduce lookup time and network load.

### Checking and Updating the Hostname

- **Display the current hostname:**
  ```bash
  hostname
  ```
- **Update the hostname:**
  ```bash
  vim /etc/hostname
  ```
  Change the content to, for example:  
  ```
  ns1.armour.local
  ```
- **Reboot to apply changes:**
  ```bash
  reboot
  ```
- **Confirm the change:**
  ```bash
  hostname
  ```

### Check DNS Resolver Configuration

Verify your DNS resolver settings:
```bash
cat /etc/resolv.conf
```
You should see lines like:
```
nameserver 8.8.8.8
nameserver 8.8.4.4
```

### Installing and Verifying BIND

- **Check if BIND is installed:**
  ```bash
  rpm -qa | grep bind
  ```
- **Install BIND and utilities:**
  ```bash
  yum install bind bind-utils -y
  ```
- **Verify installation:**
  ```bash
  rpm -qa | grep bind
  ```

---

## 7. Detailed BIND Configuration Files and Directories

BIND’s configuration is divided among multiple files and directories. Understanding them is key to managing your DNS.

### Key Files and Directories:

- **/etc/named.conf** – Main configuration file.
- **/etc/named.rfc1912.zones** – Contains default zone definitions (e.g., localhost, reverse lookup zones).
- **/etc/named.root.key** – Contains DNSSEC root trust anchors.
- **/var/named/** – The working directory for BIND; it includes:
  - `named.ca` – Root hints file.
  - `named.empty` – An empty zone file.
  - `named.localhost` – For localhost resolution.
  - `named.loopback` – For loopback resolution.
  - `dynamic/` – Directory for dynamic zone files.
  - `slaves/` – Directory for zone files received from masters (for slave configurations).

### Sample /etc/named.conf Explanation

A sample configuration includes:

```named
options {
    directory "/var/named";
    dump-file "data/cache_dump.db";
    statistics-file "data/named_stats.txt";
    memstatistics-file "data/named_mem_stats.txt";
    recursing-file "data/named.recursing";
    secroots-file "data/named.secroots";
    listen-on port 53 { 127.0.0.1; 192.168.1.28; };
    listen-on-v6 port 53 { ::1; };
    allow-query { localhost; 192.168.1.0/24; };
    allow-transfer { none; };
    recursion yes;
    managed-keys-directory "/var/named/dynamic";
};

key "ddns_key" {
    algorithm hmac-sha256;
    secret "AbCdEf1234567890RandomGeneratedKey==";
};

logging {
    channel ddns_update_log {
        file "data/ddns.log";
        severity info;
        print-time yes;
    };
    category update { ddns_update_log; };
    category update-security { ddns_update_log; };
};

zone "rs.local" IN {
    type master;
    file "dynamic/rs.local.zone";
    allow-update { key ddns_key; };
    notify yes;
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

*Explanation:*
- **options:** Sets paths, listening interfaces, recursion, and query policies.
- **key "ddns_key":** TSIG key for secure dynamic updates.
- **logging:** Configures log file and level.
- **zone "rs.local":** Defines your internal zone with dynamic update enabled.
- **include directives:** Bring in additional zone definitions and DNSSEC keys.

---

## 8. Authoritative DNS Server Configuration (Master Nameserver)

A master (primary) server holds the original zone files.

### Sample Master Nameserver Configuration

**/etc/named.conf Example for Master:**

```named
options {
    listen-on port 53 { 127.0.0.1; 192.168.1.50; };
    listen-on-v6 port 53 { ::1; };
    directory "/var/named";
    dump-file "/var/named/data/cache_dump.db";
    statistics-file "/var/named/data/named_stats.txt";
    memstatistics-file "/var/named/data/named_mem_stats.txt";
    recursing-file "/var/named/data/named.recursing";
    secroots-file "/var/named/data/named.secroots";
    allow-query { localhost; 192.168.1.0/24; };
    recursion no;  # Disable recursion on authoritative server
    dnssec-enable yes;
    dnssec-validation yes;
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

zone "." IN {
    type hint;
    file "named.ca";
};

zone "example.com" IN {
    type master;
    file "/var/named/example.com.zone";
    allow-update { none; };
};

zone "1.168.192.in-addr.arpa" IN {
    type master;
    file "/var/named/1.168.192.zone";
    allow-update { none; };
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

### Creating a Forward Zone File

Create `/var/named/example.com.zone`:

```zone
$TTL 86400
@   IN SOA ns1.example.com. admin.example.com. (
        2025031101 ; serial
        3600       ; refresh
        1800       ; retry
        604800     ; expire
        86400      ; minimum TTL
)
    IN NS ns1.example.com.
ns1 IN A 192.168.1.50
www IN A 192.168.1.100
```

### Creating a Reverse Zone File

Create `/var/named/1.168.192.zone`:

```zone
$TTL 86400
@   IN SOA ns1.example.com. admin.example.com. (
        2025031101 ; serial
        3600       ; refresh
        1800       ; retry
        604800     ; expire
        86400      ; minimum TTL
)
    IN NS ns1.example.com.
50  IN PTR ns1.example.com.
100 IN PTR www.example.com.
```

*Be sure to set proper permissions so that the BIND user (named) can write to these files:*

```bash
chown named:named /var/named/example.com.zone /var/named/1.168.192.zone
chmod 640 /var/named/example.com.zone /var/named/1.168.192.zone
```

---

## 9. Multiple-Zones Configuration

This section describes how to configure multiple zones on the same DNS server. We use two example zones: `infosec.local` and `ai.local`.

### Setup for infosec.local

1. **Edit the Zones Configuration:**  
   Edit `/etc/named.rfc1912.zones`:
   ```bash
   vim /etc/named.rfc1912.zones
   ```
   Add:
   ```named
   zone "infosec.local" IN {
       type master;
       file "forward.infosec.local";
       allow-update { none; };
   };
   ```
2. **Create the Forward Zone File:**  
   Copy an existing forward zone file and edit:
   ```bash
   cp -v /var/named/forward.armour.local /var/named/forward.infosec.local
   vim /var/named/forward.infosec.local
   ```
   Example content:
   ```zone
   $TTL 1D
   $ORIGIN infosec.local.
   @   IN SOA ns1.armour.local. root.armour.local. (
           002 ; serial
           1D  ; refresh
           1H  ; retry
           1W  ; expire
           3H  ) ; minimum
   @   IN NS ns1.armour.local.
   infosec.local. IN A 192.168.1.200
   www IN CNAME infosec.local.
   @   IN MX 10 mail.armour.local.
   @   IN TXT "v=spf1 mx a ~all"
   ftp IN A 192.168.1.112
   dev IN A 192.168.1.111
   fileserver IN A 192.168.1.110
   db IN A 192.168.1.120
   test IN A 192.168.1.130
   vpn IN A 192.168.1.140
   git IN A 192.168.1.150
   webapp IN A 192.168.1.160
   logs IN A 192.168.1.170
   ```
3. **Set Permissions:**
   ```bash
   chgrp named /var/named/forward.infosec.local
   chmod 640 /var/named/forward.infosec.local
   ```

### Setup for ai.local

1. **Edit the Zones Configuration:**  
   Edit `/etc/named.rfc1912.zones`:
   ```bash
   vim /etc/named.rfc1912.zones
   ```
   Add:
   ```named
   zone "ai.local" IN {
       type master;
       file "forward.ai.local";
       allow-update { none; };
   };
   ```
2. **Create the Forward Zone File:**  
   Copy from infosec.local:
   ```bash
   cp -v /var/named/forward.infosec.local /var/named/forward.ai.local
   vim /var/named/forward.ai.local
   ```
   Update sample content:
   ```zone
   $TTL 1D
   $ORIGIN ai.local.
   @   IN SOA ns1.armour.local. root.armour.local. (
           002 ; serial
           1D  ; refresh
           1H  ; retry
           1W  ; expire
           3H  ) ; minimum
   @   IN NS ns1.armour.local.
   ai.local. IN A 192.168.1.201
   www IN CNAME ai.local.
   @   IN MX 10 mail.armour.local.
   @   IN TXT "v=spf1 mx a -all"
   ftp IN A 192.168.1.112
   dev IN A 192.168.1.111
   fileserver IN A 192.168.1.110
   db IN A 192.168.1.120
   webapp IN A 192.168.1.160
   logs IN A 192.168.1.170
   ```
3. **Set Permissions:**
   ```bash
   chgrp named /var/named/forward.ai.local
   chmod 640 /var/named/forward.ai.local
   ```

### The $ORIGIN Directive Explained

The `$ORIGIN` directive in a zone file sets the base domain. For example:
```zone
$ORIGIN ai.local.
@   IN SOA ns1.armour.local. root.armour.local. ( ... )
www IN CNAME ai.local.
```
- Here, `@` represents `ai.local`.
- The record `www` becomes `www.ai.local.` automatically.
- If `$ORIGIN` were not set, you’d need to specify the full domain each time.

---

## 10. Master & Slave DNS Server Setup

Configuring a master (primary) and a slave (secondary) DNS server ensures redundancy.

### Master DNS Setup

- **Server Example:**  
  **IP:** 192.168.1.34  
  **Hostname:** ns1.armour.local  
- **Configuration:**  
  Refer to Section 8 above for the master nameserver config. Ensure the zone files have an `allow-transfer` clause allowing the slave’s IP.

### Slave DNS Setup

- **Server Example:**  
  **IP:** 192.168.1.38  
  **Hostname:** ns2.armour.local  
- **Configuration:**  
  Edit `/etc/named.rfc1912.zones` on the slave:
  ```named
  zone "armour.local" IN {
      type slave;
      masters { 192.168.1.34; };
      file "slaves/forward.armour.local";
  };

  zone "infosec.local" IN {
      type slave;
      masters { 192.168.1.34; };
      file "slaves/forward.infosec.local";
  };

  zone "ai.local" IN {
      type slave;
      masters { 192.168.1.34; };
      file "slaves/forward.ai.local";
  };
  ```
- **Ensure that the /var/named/slaves directory exists** and is owned by the `named` user.
- **Restart BIND** on the slave:
  ```bash
  systemctl restart named
  ```

Test DNS resolution on the slave using:
```bash
dig armour.local @192.168.1.38
dig infosec.local @192.168.1.38
dig ai.local @192.168.1.38
```

---

## 11. Firewall Configuration for DNS

Ensure DNS traffic (TCP and UDP port 53) is allowed.

### With Firewalld:

```bash
firewall-cmd --permanent --add-port=53/tcp
firewall-cmd --permanent --add-port=53/udp
firewall-cmd --reload
```

### With UFW (if applicable):

```bash
ufw allow 53/tcp
ufw allow 53/udp
```

---

## 12. Dynamic DNS (DDNS) Configuration

DDNS allows DNS records to be updated automatically when client IPs change.

### 12.1 Client-Initiated DDNS Updates

Each client runs a script using `nsupdate` to update its record.

- **Step 1: Create TSIG Key on the Server**
  ```bash
  ddns-confgen -a hmac-sha256 -k ddns_key -r /dev/urandom
  ```
  Save the generated key (e.g., secret "AbCdEf1234567890RandomGeneratedKey==").

- **Step 2: Configure BIND to Allow Secure Updates**
  In `/etc/named.conf`, add:
  ```named
  key "ddns_key" {
      algorithm hmac-sha256;
      secret "AbCdEf1234567890RandomGeneratedKey==";
  };

  zone "rs.local" IN {
      type master;
      file "dynamic/rs.local.zone";
      allow-update { key ddns_key; };
      notify yes;
  };
  ```

- **Step 3: Create the Client TSIG Key File**
  On the client (centmini.rs.local), create:
  ```bash
  mkdir -p /etc/ddns
  vi /etc/ddns/ddns.key
  ```
  Insert:
  ```plain
  key "ddns_key" {
      algorithm hmac-sha256;
      secret "AbCdEf1234567890RandomGeneratedKey==";
  };
  ```
- **Step 4: Create the nsupdate Script on the Client**

  Create `/usr/local/bin/update-ddns.sh`:
  ```
  #!/bin/bash
  KEY="/etc/ddns/ddns.key"
  DNSSERVER="192.168.1.28"
  ZONE="rs.local"
  HOSTNAME="centmini.rs.local."
  TTL="300"
  IP=$(ip addr show enp0s3 | awk '/inet / { sub(/\/.*/, "", $2); print $2; }')
  if [ -z "$IP" ]; then
      echo "Error: No IP detected on enp0s3."
      exit 1
  fi
  nsupdate -k "$KEY" <<EOF
  server $DNSSERVER
  zone $ZONE
  update delete $HOSTNAME A
  update add $HOSTNAME $TTL A $IP
  send
  EOF
  echo "DDNS update complete; $HOSTNAME now points to $IP"
  ```
  Make it executable:
  ```bash
  chmod +x /usr/local/bin/update-ddns.sh
  ```
- **Step 5: Test the Update Manually**
  ```bash
  /usr/local/bin/update-ddns.sh
  dig @192.168.1.28 centmini.rs.local
  ```
- **Step 6: Automate via Systemd Timer or DHCP Hooks**
  Create a systemd service and timer so that the script runs every 5 minutes, or set up a DHCP client hook in `/etc/dhclient-exit-hooks.d/`.

### 12.2 DHCP Server-Initiated DDNS Updates

Instead of each client updating DNS, let the DHCP server do it.

- **Edit /etc/dhcp/dhcpd.conf on the Server:**
  ```conf
  ddns-updates on;
  ddns-update-style interim;
  update-static-leases on;
  authoritative;
  
  key "ddns_key" {
      algorithm hmac-sha256;
      secret "AbCdEf1234567890RandomGeneratedKey==";
  };
  
  zone rs.local. {
      primary 192.168.1.28;
      key ddns_key;
  }
  
  zone 1.168.192.in-addr.arpa. {
      primary 192.168.1.28;
      key ddns_key;
  }
  
  subnet 192.168.1.0 netmask 255.255.255.0 {
      range 192.168.1.100 192.168.1.200;
      option domain-name "rs.local";
      option domain-name-servers 192.168.1.28;
      ddns-domainname "rs.local";
      ddns-rev-domainname "in-addr.arpa";
  }
  ```
  Restart the DHCP server:
  ```bash
  systemctl restart dhcpd
  ```
  Now, when a client receives a lease, the DHCP server automatically performs a DDNS update (no extra client configuration needed).

---

## 13. DNS Troubleshooting and Testing

- **Check BIND Status:**
  ```bash
  systemctl status named
  ```
- **Check Configuration Syntax:**
  ```bash
  named-checkconf /etc/named.conf
  named-checkconf /etc/named.rfc1912.zones
  named-checkzone rs.local /var/named/dynamic/rs.local.zone
  ```
- **Test DNS Queries with dig:**
  ```bash
  dig @192.168.1.28 centmini.rs.local
  dig example.com
  dig -x 192.168.1.100
  ```
- **Check DNS Logs:**
  Monitor `/var/named/data/ddns.log` and use `journalctl -u named`.
- **Examine DHCP Leases:**
  Look in `/var/lib/dhcp/` or `/var/lib/NetworkManager/` for DHCP lease files.

---

## 14. Conclusion and Summary

This complete DNS notes document for CentOS 9 covers all essential topics from server types and client tools to detailed configurations for caching nameservers, master/slave authoritative configurations, multiple zones (forward and reverse), and both DDNS update methods. Key points include:

- **DNS Server Types:** Understanding non-authoritative vs. authoritative, caching, and forwarding.
- **DNS Client Tools:** Using dig, nslookup, and host to query and validate configurations.
- **DNS Records:** Detailed explanation of A, AAAA, CNAME, NS, PTR, SOA, HINFO, MX, TXT.
- **BIND Configuration:** Explanation of key files and directories, with sample configurations for a caching nameserver.
- **Authoritative DNS Setup:** Master nameserver configuration with forward and reverse zone files.
- **Multiple Zones:** How to set up additional zones like infosec.local and ai.local, and the use of the $ORIGIN directive.
- **Master/Slave Setup:** Configuring a redundant environment with a primary and secondary DNS server.
- **Firewall and Testing:** How to configure firewall rules and verify DNS resolution.
- **Dynamic DNS (DDNS):** Full explanation of both client-initiated and DHCP server–initiated DDNS, with TSIG keys, scripts, automation, and integration with ISC DHCP.


--- 