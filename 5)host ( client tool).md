	**host**

**host - DNS Client Tools**

**Overview**

The host command is a simple DNS lookup utility used to convert hostnames to IP addresses and perform other DNS queries.

**Basic host Commands**

**Lookup a Domain's IP Address**

Use host to find the IP address of a domain:

```

host google.com

```

**Query Specific DNS Record Types**

**NS Record (Name Server)**

Query the name server for a domain:

```

host -t ns google.com

```

**MX Record (Mail Exchange)**

Query the mail exchange servers for a domain:

```

host -t mx google.com

```

**TXT Record (Text)**

Query the TXT records for a domain:

```

host -t txt google.com

```

**SOA Record (Start of Authority)**

Query the SOA record for a domain:

```

host -t soa google.com

```

**Specificy DNS Server**

Query the SOA record for a domain using a specific DNS server:

```

host -t soa google.com 64.6.64.6

```