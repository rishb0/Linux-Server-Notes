# DNS-Server

## DNS Server Types

### Non-Authoritative (Recursive) Nameserver

Non-authoritative nameservers are responsible for resolving DNS queries by contacting other nameservers to find the answer. They do not store original DNS records but rely on information provided by other servers. They typically perform recursion until an authoritative answer is found.

- **Caching Nameserver**  
  Caching nameservers temporarily store DNS query results to improve resolution time and reduce the load on authoritative servers. This cache is used for subsequent queries until the Time-To-Live (TTL) for a record expires.

  *Example command to test DNS resolution using a caching nameserver:*
  ```
  dig example.com @8.8.8.8
  ```

- **Forwarders Nameserver**  
  Forwarding nameservers receive DNS queries and pass them to another nameserver (usually a caching or authoritative server) for resolution. Forwarders are often used in environments where a local DNS server cannot perform external queries directly due to network security policies.
  
  *Example command to configure a forwarding nameserver:*
  ```
  echo "nameserver 8.8.8.8" >> /etc/resolv.conf
  ```

### Authoritative Nameserver

Authoritative nameservers store and provide responses for DNS queries about the domains they manage. They hold the actual DNS records for the domains and are considered the final source of information for those zones.

- **Primary Nameserver**  
  The primary nameserver (also known as the master nameserver) is the main source for DNS zone data. It is responsible for the creation, storage, and updates of the DNS records. Changes made on the primary nameserver are propagated to secondary nameservers through zone transfers.
  
  *Example of a named.conf configuration for a primary nameserver:*
  ```
  zone "example.com" {
    type master;
    file "/etc/bind/db.example.com";
  };
  ```

- **Secondary Nameserver**  
  The secondary nameserver (slave nameserver) obtains its DNS zone data from the primary nameserver and serves as a backup if the primary is unavailable. It periodically transfers the updated zone file using protocols such as AXFR.
  
  *Example of a named.conf configuration for a secondary nameserver:*
  ```
  zone "example.com" {
    type slave;
    file "/etc/bind/db.example.com";
    masters { 192.168.1.1; };
  };
  ```

# DNS Client Tools

DNS client tools are utilities used to query and troubleshoot DNS issues. They provide methods for verifying zone information, checking name resolution, and testing configurations.

## DNS Client Tools Commands

Below are the common commands used to manage and query DNS client tools:

### Check Installed BIND Packages
Use the following command to list installed BIND packages on RPM-based systems:
```
rpm -qa | grep bind
```

### Check Installed BIND Utilities
List installed `bind-utils` packages using the following command:
```
rpm -qa | grep bind-utils
```

### Install BIND Utilities
To install BIND utilities (such as dig, host, nslookup, etc.) using `yum`:
```
yum install bind-utils
```

### List Files Installed by bind-utils
To list all files installed by the `bind-utils` package:
```
rpm -ql bind-utils
```

### List Configuration Files for bind-utils
To list configuration files provided by the `bind-utils` package:
```
rpm -qc bind-utils
```

### List Documentation Files for bind-utils
To list documentation files for the `bind-utils` package:
```
rpm -qd bind-utils
```

### Common DNS Client Tools Installed with bind-utils

These are the commonly used DNS client tools included with `bind-utils`:
- **/usr/bin/delv** – DNS lookup and validation utility, providing extended security information.
- **/usr/bin/dig** – Query DNS servers for information about domains. It is versatile and commonly used for troubleshooting.
- **/usr/bin/host** – Simple utility for performing DNS lookups.
- **/usr/bin/mdig** – Multiple query version of `dig` that allows for parallel DNS queries.
- **/usr/bin/nslookup** – A legacy tool to query DNS servers; while deprecated in some systems, it is still widely available for compatibility.
- **/usr/bin/nsupdate** – Utility for performing dynamic DNS updates, allowing automated changes to zone files.

# DNS Records

DNS records define how domain names are mapped to IP addresses and other resources. Each record type has a specific purpose and format.

### A (Address Mapping) Records
- Specifies an IPv4 address for a given host.
- *Example:*
  ```
  example.com ➜ 192.168.1.1
  ```

### AAAA (IPv6 Address) Records
- Specifies an IPv6 address for a given host.
- *Example:*
  ```
  example.com ➜ 2001:0db8::ff00:0042:8329
  ```

### CNAME (Canonical Name) Records
- Specifies an alias for another domain name. CNAME records point one domain name to another, meaning that the DNS lookup will continue by retrying the lookup with the new name.
- *Example:*
  ```
  wp.armourinfosec.com ➜ armourinfosec.wordpress.com ➜ 20.20.20.20
  ```

### NS (Name Server) Records
- Specifies an authoritative name server for a given domain. NS records delegate the DNS resolution to the specified server.
- *Example:*
  ```
  example.com ➜ ns1.example.com
  ```

### PTR (Pointer) Records
- Used for reverse DNS lookups, which map an IP address back to a hostname. This is essential for verifying the legitimacy of a host and is often used in email server configurations.
- *Example:*
  ```
  192.168.1.1 ➜ example.com
  ```

### SOA (Start of Authority) Records
- Specifies authoritative information about a DNS zone, including the primary nameserver, the email of the domain administrator (with a dot instead of @), and various timers for zone maintenance (serial number, refresh, retry, expire, and minimum TTL).
- *Example:*
  ```
  example.com. IN SOA ns1.example.com. admin.example.com. (
             2025031101 ; serial
             3600       ; refresh
             1800       ; retry
             1209600    ; expire
             86400      ; minimum TTL
  )
  ```

### HINFO (Host Information) Records
- Provides information about a host's hardware and operating system. Although not widely used today due to privacy concerns, it was originally designed to help with system identification.
- *Example:*
  ```
  example.com. IN HINFO "Intel i7" "Linux"
  ```

### MX (Mail Exchanger) Records
- Specifies the mail exchange server responsible for receiving email messages on behalf of a domain. The record includes a priority value that determines the order in which servers are used (a lower value indicates a higher preference).
- *Example:*
  ```
  example.com ➜ mail.example.com (priority 10)
  ```

### TXT (Text) Records
- Holds arbitrary text data for a domain. TXT records are often used for various purposes such as domain ownership verification, SPF (Sender Policy Framework) to prevent email spoofing, or DKIM (DomainKeys Identified Mail) configurations.
- *Example:*
  ```
  example.com. IN TXT "v=spf1 include:_spf.google.com ~all"
  ```