**nslookup**

**nslookup - DNS Client Tools**

The nslookup (Name Server Lookup) command is a network administration tool for querying the Domain Name System (DNS) to obtain domain name or IP address mapping and other DNS records.

**Basic nslookup Commands**

**Lookup a Domain's IP Address**

Use nslookup to find the IP address of a domain:

```

nslookup google.com

```

**Lookup a Domain Using a Specific DNS Server**

Query a domain using a specific DNS server:

```

nslookup google.com 64.6.64.6

```

**Query Specific DNS Record Types**

**A Record (IPv4)**

Query the A record (IPv4 address) of a domain:

```

nslookup -type=A google.com

```

**AAAA Record (IPv6)**

Query the AAAA record (IPv6 address) of a domain:

```

nslookup -type=AAAA google.com

```

**NS Record (Name Server)**

Query the name server for a domain:

```

nslookup -type=NS google.com

```

**MX Record (Mail Exchange)**

Query the mail exchange servers for a domain:

```

nslookup -type=MX google.com

```

**SOA Record (Start of Authority)**

Query the SOA record for a domain:

```

nslookup -type=SOA google.com

```

**TXT Record (Text)**

Query the TXT records for a domain:

```

nslookup -type=TXT google.com

```

**Other Useful Lookups**

**Lookup a Subdomain**

Query a specific subdomain:

```

nslookup www.armourinfosec.com

```

**Query All Available Records**

Retrieve all available DNS records for a domain:

```

nslookup -type=ANY google.com

```

**Interactive nslookup Mode**

You can use nslookup in interactive mode to perform multiple queries without reopening the tool each time:

**Start Interactive Mode**

Launch nslookup without any arguments:

```

nslookup

```

**Example Interactive Commands:**

1. Query a specific domain:

   ```

   facebook.com

   ```

2. Set the query type to NS records:

   ```

   set type=ns

   ```

   ```

   facebook.com

   ```

3. Set the query type to TXT records:

   ```

   set type=txt

   ```

   ```

   facebook.com

   ```

4. Use a specific DNS server:

   ```

   server 64.6.64.6

   ```
