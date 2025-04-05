# dig

**dig - DNS Client Tools **

**Overview**

The dig (Domain Information Groper) command is a powerful DNS lookup tool used to query DNS servers for information about host addresses, mail exchanges, name servers, and other DNS records.

**Basic dig Commands**

**Perform a Basic DNS Lookup**

Use dig without any argument to get 13 root Server:

```

dig

```

**Lookup a Specific Domain**

Query DNS records for a specific domain:

```

dig google.com

```

**Query a Specific DNS Server**

Query google.com using a specific DNS server:

```

dig google.com @64.6.64.6

```

- Output:

```

; <<>> DiG 9.16.23-RH <<>> google.com @64.6.64.6

;; global options: +cmd

;; Got answer:

;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 31294

;; flags: qr rd ra; QUERY: 1, ANSWER: 6, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:

; EDNS: version: 0, flags:; udp: 4096

;; QUESTION SECTION:

;google.com.                    IN      A

;; ANSWER SECTION:

google.com.             284     IN      A       142.251.12.138

google.com.             284     IN      A       142.251.12.113

google.com.             284     IN      A       142.251.12.100

google.com.             284     IN      A       142.251.12.101

google.com.             284     IN      A       142.251.12.102

google.com.             284     IN      A       142.251.12.139

;; Query time: 68 msec

;; SERVER: 64.6.64.6#53(64.6.64.6)

;; WHEN: Tue Mar 11 17:32:08 IST 2025

;; MSG SIZE  rcvd: 135

```

- **Explanation:**

- **HEADER Section:**

  - **opcode:** QUERY - The operation code for the query.

  - **status:** NOERROR - Indicates the query was successful.

  - **id:** 31294 - A unique identifier for the query.

- **QUESTION SECTION:**

  - Lists the domain queried: `google.com` with type `A` (IPv4 address).

- **ANSWER SECTION:**

  - Provides the IP addresses for `google.com`:

    - 142.251.12.138

    - 142.251.12.113

    - 142.251.12.100

    - 142.251.12.101

    - 142.251.12.102

    - 142.251.12.139

- **Query time:** 68 msec - Time taken to receive the response.

- **SERVER:** 64.6.64.6 - The DNS server used for the query.

- **WHEN:** Tue Mar 11 17:32:08 IST 2025 - The date and time of the query.

- **MSG SIZE rcvd:** 135 - The size of the response message in bytes.

**Query Specific DNS Record Types**

**A Record (IPv4)**

Query the IPv4 address of a domain:

```

dig google.com A

```

or:

```

dig A google.com

```

**AAAA Record (IPv6)**

Query the IPv6 address of a domain:

```

dig google.com AAAA

```

or:

```

dig AAAA google.com

```

**NS Record (Name Server)**

Query the name server for a domain:

```

dig google.com NS

```

or:

```

dig ns google.com

```

**MX Record (Mail Exchange)**

Query the mail exchange server for a domain:

```

dig google.com MX

```

**SOA Record (Start of Authority)**

Query the SOA record for a domain:

```

dig google.com SOA

```

**TXT Record (Text)**

Query TXT records for a domain:

```

dig google.com TXT

```

**ANY Record (All available records)**

Retrieve all available DNS records for a domain:

```

dig google.com ANY

```

**HINFO Record (Host Information)**

Query the HINFO record for a domain:

```

dig armourinfosec.com HINFO

```

**Shortened Output**

**Short Answer Format**

Display a simplified output:

```

dig google.com +short

```

**A Record in Short Format**

```

dig armourinfosec.com A +short

```

**AAAA Record in Short Format**

```

dig armourinfosec.com AAAA +short

```

**NS Record in Short Format**

```

dig armourinfosec.com NS +short

```

**MX Record in Short Format**

```

dig armourinfosec.com MX +short

```

**ANY Record in Short Format**

```

dig google.com ANY +short

```

**Customizing Output**

**Suppress All Output**

Suppress all output:

```

dig google.com +noall

```

**Display Only the Answer Section**

```

dig google.com +noall +answer

```

**Display the Question and Answer Section**

```

dig google.com +noall +answer +question

```

**Fine-tune Output Display**

Suppress comments, questions, authority, additional, and stats sections:

```

dig google.com +nocomments +noquestion +noauthority +noadditional +nostats

```

**Bulk DNS Lookup**

**Create a File with Domains**

Create a file named `domain_list.txt`:

```

vim domain_list.txt

```

Example content:

```

google.com

facebook.com

armourinfosec.com

youtube.com

```

**Query Multiple Domains from a File**

Use `dig` to query all domains listed in the file:

```

dig -f domain_list.txt

```

**Shortened Output for Multiple Domains**

Get a simplified output for all domains:

```

dig -f domain_list.txt +short

```

**Save Output to a File**

Save the output to a file:

```

dig -f domain_list.txt +short > ip_add_list.txt

```

**Reverse DNS Lookup**

**Reverse Lookup (PTR Record)**

Find the hostname associated with an IP address:

```

dig -x 8.8.8.8

```

or:

```

dig -x 8.8.8.8 PTR

```

**Shortened Reverse Lookup Output**

```

dig -x 8.8.8.8 +short

```

**Reverse Lookup Using Bulk IPs**

**Create a File with IP Addresses**

Create a file named `ip_add_list.txt`:

```

vim ip_add_list.txt

```

Example content:

```

8.8.8.8

64.6.64.6

8.8.4.4

1.1.1.1

20.20.20.20

```

**Generate -x Format Using awk**

Convert IPs to `-x` format using `awk`:

```

awk '$0="-x " $0' ip_add_list.txt > ip_add_list2.txt

```

**Generate -x Format Using sed**

Convert IPs to `-x` format using `sed`:

```

cat ip_add_list.txt | sed -e "s/.*/-x &/" > ip_add_list3.txt

```

**Query Reverse DNS for Multiple IPs**

Perform reverse lookups using the modified file:

```

dig -f ip_add_list2.txt +short

```

**Query Reverse DNS from an IP List File**

```

dig -f ip.txt +short

```
