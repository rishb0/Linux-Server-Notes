## 1. Client-Initiated Updates (Current Setup)

- **How It Works:**  
  Each client runs a script that uses nsupdate to update its A record with the DNS server.  
- **Advantages:**  
  It gives each client control over updating its own record.  
- **Disadvantages:**  
  You have to deploy and maintain the script on every client (or use hooks in the DHCP client).

---
# Comprehensive DDNS Setup Guide on CentOS 9

## Table of Contents
1. [Preparation on Server and Client](#preparation)
2. [Server-Side Configuration](#server-side-configuration)
   - [Generate TSIG Key](#generate-tsig-key)
   - [Create Directory for Dynamic Zone Files](#create-directory-for-dynamic-zone-files)
   - [Edit named.conf](#edit-namedconf)
   - [Create Initial Zone File](#create-initial-zone-file)
   - [Set Permissions and Start BIND](#set-permissions-and-start-bind)
3. [Client-Side Configuration](#client-side-configuration)
   - [Create TSIG Key File](#client-tsig-key-file)
   - [Create the nsupdate Script](#create-the-nsupdate-script)
   - [Test the Update Manually](#test-the-update-manually)
4. [Automation and DHCP Hooks](#automation-and-dhcp-hooks)
5. [What Happens When the IP Changes](#what-happens-when-the-ip-changes)
6. [Summary](#summary)

---

## 1. Preparation on Server and Client <a name="preparation"></a>

**1.1 Update the System**

On **both** the server and the client, run:

```bash
dnf update -y
```

*This ensures you have the latest packages and security updates.*

**1.2 Install Required Packages**

On the **SERVER** (to install BIND and utilities):

```bash
dnf install -y bind bind-utils policycoreutils-python-utils
```

On the **CLIENT** (ensure nsupdate is available; it usually comes with bind-utils):

```bash
dnf install -y bind-utils
```

---

## 2. Server-Side Configuration <a name="server-side-configuration"></a>

We are configuring the DNS server (centos.rs.local with IP 192.168.1.28) to accept secure dynamic updates for the zone `rs.local`.

### 2.1 Generate TSIG Key <a name="generate-tsig-key"></a>

TSIG keys secure updates so that only trusted clients can update DNS records.

On the **SERVER**, run:

```bash
ddns-confgen -a hmac-sha256 -k ddns_key -r /dev/urandom
```

This command will output a snippet similar to:

```bash
key ddns_key {
    algorithm hmac-sha256;
    secret "AbCdEf1234567890RandomGeneratedKey==";
};
```

**Note:** Copy the secret (the string between the quotes) because you need the same on the client.

---

### 2.2 Create Directory for Dynamic Zone Files <a name="create-directory-for-dynamic-zone-files"></a>

We store our dynamic zone files and journals in a dedicated directory.

```bash
mkdir -p /var/named/dynamic
chown -R named:named /var/named/dynamic
chmod 770 /var/named/dynamic
```

*Explanation:*
- **mkdir -p** creates the directory.
- **chown** and **chmod** ensure that the BIND user (`named`) can write to this folder.

---

### 2.3 Edit named.conf <a name="edit-namedconf"></a>

Open **/etc/named.conf** with your editor (e.g., `vi /etc/named.conf`). Replace or add the following content, with comments explaining each section:

```named
/*
    /etc/named.conf – Main Configuration for BIND
*/

options {
    # Set the working directory for zone files.
    directory "/var/named";

    # Files for caching and statistics.
    dump-file "data/cache_dump.db";
    statistics-file "data/named_stats.txt";
    memstatistics-file "data/named_mem_stats.txt";

    # Listen on localhost and the server's IP.
    listen-on port 53 { 127.0.0.1; 192.168.1.28; };
    listen-on-v6 port 53 { ::1; };

    # Allow queries from localhost and the local network.
    allow-query { localhost; 192.168.1.0/24; };

    # Disable zone transfers by default.
    allow-transfer { none; };

    # Enable recursion (for local caching).
    recursion yes;

    # Managed keys directory for dynamic updates.
    managed-keys-directory "/var/named/dynamic";
};

/* Define the TSIG key for secure dynamic updates. */
key "ddns_key" {
    algorithm hmac-sha256;
    secret "AbCdEf1234567890RandomGeneratedKey==";  // Replace with your generated secret.
};

/* Logging configuration for update events (optional). */
logging {
    channel ddns_update_log {
        file "data/ddns.log";
        severity info;
        print-time yes;
    };
    category update { ddns_update_log; };
    category update-security { ddns_update_log; };
};

/* Define the zone rs.local (our internal DNS domain). */
zone "rs.local" IN {
    type master;
    file "dynamic/rs.local.zone";   // The zone file path.
    allow-update { key ddns_key; };  // Allow updates only if authenticated with our key.
    notify yes;                     // Enable notifications if you have secondary servers.
};
```

*Explanation:*
- **options:** Sets up working directories, listening interfaces, query permissions, and recursion.
- **key "ddns_key":** This section defines our TSIG key used to authenticate updates.
- **zone "rs.local":** This configures our domain. The `allow-update` line tells BIND to only allow dynamic updates from a client that presents the correct TSIG key.

---

### 2.4 Create Initial Zone File <a name="create-initial-zone-file"></a>

Create the zone file for `rs.local`.

```bash
vi /var/named/dynamic/rs.local.zone
```

Paste in the following:

```zone
$TTL 3600
@       IN      SOA     centos.rs.local. admin.rs.local. (
                        2023101001 ; Serial (update this when changes occur)
                        3600       ; Refresh (in seconds)
                        1800       ; Retry (in seconds)
                        604800     ; Expire (in seconds)
                        86400      ; Minimum TTL (in seconds)
)
        IN      NS      centos.rs.local.

centos IN      A       192.168.1.28
```

*Explanation:*
- **$TTL 3600:** Sets the default time-to-live for records.
- **SOA record:** Contains the Start of Authority; it defines the primary nameserver and email (admin.rs.local, note the dot replacing @).
- **NS record:** Declares the authoritative DNS server.
- **A record for "centos":** Maps centos.rs.local to the IP 192.168.1.28.

---

### 2.5 Set File Permissions and Start BIND <a name="set-permissions-and-start-bind"></a>

Ensure BIND can write to the zone file:

```bash
chown named:named /var/named/dynamic/rs.local.zone
chmod 660 /var/named/dynamic/rs.local.zone
```

Now, start and enable the BIND service:

```bash
systemctl enable --now named
```

Check the service status with:

```bash
systemctl status named
```

*If there are any errors, review the logs using `journalctl -u named` and verify syntax with `named-checkconf /etc/named.conf` and `named-checkzone rs.local /var/named/dynamic/rs.local.zone`.*

---

## 3. Client-Side Configuration <a name="client-side-configuration"></a>

On the **CLIENT** (centmini.rs.local), we will set up automated DDNS updates using the `nsupdate` tool.

### 3.1 Create the TSIG Key File <a name="client-tsig-key-file"></a>

On the CLIENT, create a directory and file for the key:

```bash
mkdir -p /etc/ddns
vi /etc/ddns/ddns.key
```

Insert the following into the file (use the exact secret as generated on the server):

```plain
key "ddns_key" {
    algorithm hmac-sha256;
    secret "AbCdEf1234567890RandomGeneratedKey==";
};
```

Secure the file:

```bash
chmod 600 /etc/ddns/ddns.key
```

---

### 3.2 Create the nsupdate Script <a name="create-the-nsupdate-script"></a>

Create a script that uses `nsupdate` to update the DNS record for centmini.rs.local.

```bash
vi /usr/local/bin/update-ddns.sh
```

Enter the following content:

```bash
#!/bin/bash
#
# Script: update-ddns.sh
# Purpose: Dynamically update the DNS record for centmini.rs.local
#

# Path to the TSIG key file (same as on the server).
KEY="/etc/ddns/ddns.key"

# Our DNS server's IP.
DNSSERVER="192.168.1.28"

# Domain (zone) to be updated.
ZONE="rs.local"

# Fully qualified domain name for the client. Note the trailing dot!
HOSTNAME="centmini.rs.local."

# Record TTL (Time To Live in seconds).
TTL="300"

# Get current IP address from interface eth0.
# (Change eth0 if your interface name is different.)
IP=$(ip addr show enp0s3 | awk '/inet / { sub(/\/.*/, "", $2); print $2; }')

if [ -z "$IP" ]; then
    echo "Error: No IP address detected on enp0s3."
    exit 1
fi

# Use nsupdate to send a DNS update request.
nsupdate -k "$KEY" <<EOF
server $DNSSERVER
zone $ZONE
update delete $HOSTNAME A
update add $HOSTNAME $TTL A $IP
send
EOF

echo "DDNS update complete; $HOSTNAME is now set to $IP"
```

Make the script executable:

```bash
chmod +x /usr/local/bin/update-ddns.sh
```

*Explanation:*
- The script finds the client’s current IP.
- It then calls `nsupdate` with the TSIG key to connect to the DNS server and update the A record.
- The script first deletes the old A record (if present) and then adds a new A record with the current IP.

---

### 3.3 Test the Update Manually <a name="test-the-update-manually"></a>

Run the update script on the CLIENT:

```bash
/usr/local/bin/update-ddns.sh
```

Then, from either the CLIENT or SERVER, verify the DNS record:

```bash
dig @192.168.1.28 centmini.rs.local
```

The output should show an A record for `centmini.rs.local` with the client’s current IP.

---

## 4. Automation and DHCP Hooks <a name="automation-and-dhcp-hooks"></a>

### 4.1 Using Systemd Timer (Recommended)

To update DDNS automatically every five minutes, create a systemd service and timer on the CLIENT.

Create the service file:

```bash
vi /etc/systemd/system/ddns-update.service
```

Insert:

```ini
[Unit]
Description=Dynamic DNS Update Service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/update-ddns.sh
```

Now create a timer file:

```bash
vi /etc/systemd/system/ddns-update.timer
```

Insert:

```ini
[Unit]
Description=Run DDNS Update every 5 minutes

[Timer]
OnBootSec=1min
OnUnitActiveSec=5min

[Install]
WantedBy=timers.target
```

Reload systemd and start the timer:

```bash
systemctl daemon-reload
systemctl enable --now ddns-update.timer
```

### 4.2 DHCP Client Hook (Alternate Method)

If the client receives its IP via DHCP, you can use a dhclient hook to update DNS on lease renewals.

Create a hook file:

```bash
vi /etc/dhclient-exit-hooks.d/ddns-update
```

Insert:

```bash
#!/bin/bash
if [ "$reason" = "BOUND" ] || [ "$reason" = "RENEW" ]; then
    /usr/local/bin/update-ddns.sh
fi
```

Make the hook executable:

```bash
chmod +x /etc/dhclient-exit-hooks.d/ddns-update
```

*This hook will run every time DHCP assigns or renews an IP.*

---

## 5. What Happens When the IP Changes <a name="what-happens-when-the-ip-changes"></a>

- **Dynamic IPs (DHCP):**  
  When the client's IP changes (either through DHCP renewal or manually), the update script is executed automatically (via the systemd timer or dhclient hooks). It sends an authenticated update to the DNS server, which then overwrites the previous A record in the zone. Clients querying for `centmini.rs.local` will receive the new IP.

- **Static or Manual Changes:**  
  If the IP is changed manually, you can re-run the script manually or rely on the automation timer to update the record eventually.

---

## 6. Summary <a name="summary"></a>

### On the DNS Server (centos.rs.local – 192.168.1.28):

1. **Update and Install Packages:**  
   Ensure the system is updated and install BIND and required utilities.

2. **Generate TSIG Key:**  
   Use `ddns-confgen` to create a secure key that both server and client will use.

3. **Create Directory for Dynamic Zones:**  
   Prepare `/var/named/dynamic` for storing the zone file and journal.

4. **Configure named.conf:**  
   - Define global options (directories, interfaces, logging, recursion).
   - Add the TSIG key definition.
   - Configure the zone “rs.local” with `allow-update { key ddns_key; }`.

5. **Create the Zone File:**  
   Write the initial zone data in `/var/named/dynamic/rs.local.zone` and set proper permissions.

6. **Start and Verify BIND:**  
   Enable and start the named service checking for errors.

### On the DNS Client (centmini.rs.local):

1. **Install bind-utils:**  
   Ensure tools like `nsupdate` are installed.

2. **Create TSIG Key File:**  
   Create `/etc/ddns/ddns.key` with the same key used on the server.

3. **Write the nsupdate Script:**  
   Create `/usr/local/bin/update-ddns.sh` that removes any existing A record and adds a new record with the current IP.

4. **Test the Update:**  
   Run the script and use `dig` to verify that `centmini.rs.local` now points to the current IP.

5. **Automate Updates:**  
   Use systemd timers or DHCP client hooks to run the update script automatically on IP changes.

By following these detailed, step-by-step instructions along with the explanations, you should be able to successfully configure DDNS on your CentOS 9 environment from scratch. This guide covers both the server and client setup, ensuring that when the client’s IP changes, the DNS record is updated automatically. Happy configuring!