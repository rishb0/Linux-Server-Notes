1️⃣ Types of DNS Servers in Linux

Linux allows configuring different types of DNS servers based on their function and role. These include:

|                              |                                                                                                     |
| ---------------------------- | --------------------------------------------------------------------------------------------------- |
| DNS Server Type              | Description                                                                                         |
| Caching-Only DNS Server      | Caches previously resolved queries to speed up DNS resolution for clients.                          |
| Forwarding DNS Server        | Forwards queries to upstream DNS servers (like Google 8.8.8.8).                                     |
| Authoritative DNS Server     | Stores actual DNS records for domains (can be primary or secondary).                                |
| Recursive DNS Server         | Resolves DNS queries step-by-step, contacting root, TLD, and authoritative servers.                 |
| Master (Primary) DNS Server  | Holds the original, editable zone files for a domain.                                               |
| Slave (Secondary) DNS Server | A backup DNS server that gets records from the primary (read-only).                                 |
| Split-Horizon DNS Server     | Provides different DNS responses depending on the requester’s network (internal vs external users). |
| Dynamic DNS (DDNS) Server    | Allows automatic DNS updates when IP addresses change.                                              |
