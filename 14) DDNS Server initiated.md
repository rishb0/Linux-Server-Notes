# Automating DDNS Entry Creation for DHCP Clients

In Windows Server 2022, the integrated DHCP service can automatically update DNS records when a client receives an IP. In Linux environments (using ISC DHCP and BIND), you can achieve similar functionality.

There are two main approaches:

---

## 1. Client-Initiated Updates (Current Setup)

- **How It Works:**  
  Each client runs a script that uses nsupdate to update its A record with the DNS server.  
- **Advantages:**  
  It gives each client control over updating its own record.  
- **Disadvantages:**  
  You have to deploy and maintain the script on every client (or use hooks in the DHCP client).

*This method is what you’ve already set up by running an update script on the client.*

---

## 2. DHCP Server-Initiated (Integrated DDNS Updates)

This approach uses the DHCP server to update DNS records automatically as it assigns IPs to clients. It works similarly to Windows Server and involves configuring the DHCP server and DNS server to communicate updates using TSIG keys.

### How to Set It Up:

#### A. Configure the DHCP Server (ISC DHCPd)

1. **Edit the DHCP Configuration File** (typically `/etc/dhcp/dhcpd.conf`):
   
   Add lines similar to the following:

   ```conf
   ddns-updates on;
   ddns-update-style interim;
   update-static-leases on;

   key "ddns_key" {
       algorithm hmac-sha256;
       secret "AbCdEf1234567890RandomGeneratedKey==";  // Use your generated key
   };

   zone rs.local. {
       primary 192.168.1.28;  // DNS server IP (BIND)
       key ddns_key;
   }

   zone 1.168.192.in-addr.arpa. {
       primary 192.168.1.28;  // For reverse zone updates if needed
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

2. **Explanation of Key Options:**

   - `ddns-updates on;`  
     Enables dynamic DNS updates from the DHCP server.

   - `ddns-update-style interim;`  
     This tells ISC DHCP to use a modern format for updates.  

   - `update-static-leases on;`  
     Ensures that even leases with fixed addresses will update DNS.

   - `key "ddns_key" { ... }`  
     This defines the TSIG key that both DHCP and DNS will use for secure updates.

   - `zone rs.local. { ... }`  
     This block instructs the DHCP server to update the DNS zone (rs.local) on the specified primary server (our DNS server) using the provided key.

   - The subnet declaration includes options for the domain name and DNS servers so that clients know what domain they belong to, and it also configures the DHCP server to update DNS with the appropriate zone names.

3. **Restart the DHCP Service:**

   ```bash
   sudo systemctl restart dhcpd
   ```

#### B. Configure the DNS Server (BIND)

1. **Ensure Your DNS Zone (rs.local) is Configured to Accept Dynamic Updates Using the Same TSIG Key.**  
   
   In your BIND configuration (in `/etc/named.conf` or your zone file), you should already have something like:

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
   }
   ```

2. **Restart BIND if Necessary:**

   ```bash
   sudo systemctl restart named
   ```

#### C. How It All Works Together:

- **When a client requests an IP from the DHCP server:**
  - The DHCP server assigns an IP to the client.
  - Immediately after the assignment, DHCP sends a dynamic update to the DNS server using the TSIG key.
  - The DNS server receives and applies the update, creating or updating the A record for that client.

- **No Need for a Client-Side Script:**  
  Because the update is coming from the DHCP server itself, every client that gets an IP from the DHCP server will have its DNS record updated automatically—much like Windows Server’s integrated DDNS solution.

---

## Summary

- **Client-Initiated Update:**  
  Manual (or via a hook/script) update using nsupdate on each client.  
  *You’re already familiar with this method.*

- **DHCP Server-Initiated Update:**  
  Configure ISC DHCP to use `ddns-updates on` and securely update DNS records in BIND via TSIG keys.  
  *This automates the process so that all DHCP-assigned clients receive A (and optionally PTR) records without needing individual scripts.*

Using the DHCP server to perform DDNS updates is typically the preferred method in a network where all clients receive IP addresses from the same DHCP server. This method reduces configuration effort on each client and centralizes the dynamic update process.

---
