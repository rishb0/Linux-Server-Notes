**Forwarders-Nameserver**

---

**Forwarders Nameserver**

**Edit Configuration for Forwarders**

Edit the main configuration file:

```
vim /etc/bind/named.conf.options
```

**Sample Configuration for Forwarders**

Here's an example `named.conf.options` configuration file for setting up forwarders:

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
    
    forwarders {
        8.8.8.8;
        8.8.4.4;
    };

    dnssec-validation auto;
    
    auth-nxdomain no;    # conform to RFC1035
    
    // Recommended options for improved security
    version none;
    allow-transfer { none; };
};
```

---

**Explanation of Forwarders**

- **forwarders { 8.8.8.8; 8.8.4.4; };**

  - This section tells the DNS server to forward queries it can't resolve locally to Google's public DNS servers (8.8.8.8 and 8.8.4.4).

  - Forwarders improve resolution time and efficiency by caching frequently used queries.

**Restart Services**

**Restart BIND Service**

```
systemctl restart bind9
```

---

**Test DNS Queries**

Use `dig` to test DNS queries:

```
dig google.com
```

---

**Capture DNS Traffic Using tcpdump**

Capture DNS traffic on port 53 using `tcpdump`:

```
tcpdump -t udp
```

```
tcpdump -i enp0s3 port 53
```

```
tcpdump -n udp port 53
```

```
tcpdump -t udp -i enp0s3 port 53
```

```
tcpdump -n -t udp -i enp0s3 port 53
```

This setup configures a caching DNS server with forwarding enabled — useful for speeding up DNS resolution and reducing external DNS queries! 🚀