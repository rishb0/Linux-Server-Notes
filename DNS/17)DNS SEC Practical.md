# Practical DNSSEC Configuration Guide for CentOS 9

This guide walks you through the steps to deploy DNSSEC on CentOS 9 using BIND. It provides hands-on instructions on generating keys, signing zone files, updating configuration, and testing DNSSEC validation. Follow each step carefully, and refer to the theoretical section above for details about significance and record types.

---

## Table of Contents

1. [Prerequisites and Environment Setup](#prerequisites-and-environment-setup)
2. [Generating DNSSEC Keys](#generating-dnssec-keys)
3. [Preparing and Signing a Zone File](#preparing-and-signing-a-zone-file)
4. [Configuring BIND for DNSSEC](#configuring-bind-for-dnssec)
5. [Reloading and Testing DNSSEC](#reloading-and-testing-dnssec)
6. [Key Management and Rollovers (Overview)](#key-management-and-rollovers-overview)
7. [Troubleshooting and Verification](#troubleshooting-and-verification)
8. [Conclusion](#conclusion)

---

## 1. Prerequisites and Environment Setup

Before you begin, ensure that:

- You are running CentOS 9 with BIND installed.
- You have basic knowledge of Linux command-line tools.
- You are working with a zone file (e.g., for the domain `example.com`) that you want to sign.

### Install Required Packages

Install BIND and its DNSSEC utilities (usually included with BIND):

```bash
dnf install bind bind-utils -y
```

Verify installation by checking package versions:

```bash
named -v
```

---

## 2. Generating DNSSEC Keys

DNSSEC relies on two types of keys: the Key Signing Key (KSK) and the Zone Signing Key (ZSK). The KSK signs the DNSKEY record set, and the ZSK signs the rest of the zone data.

### 2.1 Generate the Zone Signing Key (ZSK)

Generate a ZSK with a lower key size (example uses 1024-bit for demonstration, but 2048-bit or higher is recommended for production):

```bash
cd /var/named
dnssec-keygen -a RSASHA256 -b 1024 -n ZONE example.com
```

This command creates two files:
- Kexample.com.+008+<key_id>.key – public key file
- Kexample.com.+008+<key_id>.private – private key file

### 2.2 Generate the Key Signing Key (KSK)

Generate a KSK with a larger key size (example uses 2048-bit):

```bash
dnssec-keygen -a RSASHA256 -b 2048 -n ZONE -f KSK example.com
```

Again, two files are generated:
- Kexample.com.+008+<key_id>.key (KSK public)
- Kexample.com.+008+<key_id>.private (KSK private)

### Note:

Keep your private keys secure and restrict access (for example, set proper file permissions).

---

## 3. Preparing and Signing a Zone File

Assume you already have an unsigned zone file for `example.com` (typically located in `/var/named/example.com.zone`). We will sign this zone file using the generated keys.

### 3.1 Prepare the Unsigned Zone File

For example, your `/var/named/example.com.zone` file may look like:

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

Ensure the file is up-to-date with a current serial number.

### 3.2 Sign the Zone

Use the `dnssec-signzone` utility to sign the zone. It incorporates the DNSKEY records and produces a signed zone file along with the RRSIG records.

Run the following command from the directory containing your zone file (e.g., `/var/named`):

```bash
dnssec-signzone -A -3 $(head -c 1000 /dev/random | sha1sum | awk '{print $1}') -N INCREMENT -o example.com -t example.com.zone
```

**Parameters Explanation:**
- `-A`: Automatically add the DNSKEY RRset.
- `-3 [salt]`: Generates a random salt for NSEC3 if needed (for privacy protection). The command above uses a quick random generation.
- `-N INCREMENT`: Instructs dnssec-signzone to automatically increment the serial number.
- `-o example.com`: Specifies the origin (domain name).
- `-t`: Trust that the zone file is formatted correctly.
- `example.com.zone`: Name of the unsigned zone file.

After running, a new file is created, typically named `example.com.zone.signed`.

---

## 4. Configuring BIND for DNSSEC

Now that you have a signed zone, update your BIND configuration to use it.

### 4.1 Update /etc/named.conf

Edit your `/etc/named.conf` to refer to the signed zone file. For instance:

```named
options {
    directory "/var/named";
    dump-file "data/cache_dump.db";
    statistics-file "data/named_stats.txt";
    memstatistics-file "data/named_mem_stats.txt";
    allow-query { localhost; 192.168.1.0/24; };
    listen-on port 53 { 127.0.0.1; 192.168.1.50; };
    recursion yes;
    dnssec-enable yes;
    dnssec-validation yes;
};

zone "example.com" IN {
    type master;
    file "example.com.zone.signed";  // Use the signed zone file
    allow-update { none; };
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

### 4.2 Reload BIND

Restart or reload BIND to apply the DNSSEC configuration:

```bash
systemctl restart named
```

Ensure there are no syntax errors by running:

```bash
named-checkconf /etc/named.conf
named-checkzone example.com /var/named/example.com.zone.signed
```

---

## 5. Reloading and Testing DNSSEC

After reloading, test that the DNSSEC signatures are being served correctly.

### 5.1 Using dig to Test DNSSEC

Query the signed zone with DNSSEC enabled:

```bash
dig example.com +dnssec
```

Examine the output:
- Look for **RRSIG** records in the ANSWER section.
- Validate that the `DNSKEY` and `RRSIG` records appear as expected.

For a short, validating answer:

```bash
dig example.com +short +dnssec
```

### 5.2 Validating the Signatures

You can use the `dnssec-verify` tool to check the integrity of your signed zone. From within the zone directory:

```bash
dnssec-verify -o example.com example.com.zone.signed
```

This checks that the signatures and keys are valid and correctly chained.

---

## 6. Key Management and Rollovers (Overview)

DNSSEC keys (KSK and ZSK) have a limited lifetime and should be rolled over periodically.

### Best Practices for Key Management:
- **Secure Storage:**  
  Keep private keys in secure directories with restricted permissions.
- **Regular Rollovers:**  
  Plan a periodic schedule (e.g., annually for KSK, more frequently for ZSK) to generate new keys and re-sign the zone.
- **Automated Tools:**  
  Use scripts or DNS management systems that can automate key rollovers and re-signing of zone files.
- **Monitor DNSSEC State:**  
  Regularly verify your zone with `dnssec-verify` and monitor logs for errors.

---

## 7. Troubleshooting and Verification

### Logging and Debugging
- **Check Service Status:**
  ```bash
  systemctl status named
  ```
- **View BIND Logs:**
  Logs are typically found in `/var/log/messages` or via `journalctl -u named`.
- **Verify DNSSEC Records:**
  Use `dig +dnssec` on various queries to ensure RRSIG, DNSKEY, and DS records are present and valid.
- **Packet Capture:**
  Use tcpdump to examine DNS traffic on port 53:
  ```bash
  tcpdump -n -i eth0 port 53
  ```

### Common Errors
- **Signature Verification Failures:**  
  Ensure time synchronization is correct between the DNS server and validating resolvers.
- **Zone Transfer Issues:**  
  Check zone file permissions and file paths.
- **Key Expiry/Rollover Problems:**  
  Make sure to update DS records with the parent zone after a key rollover.

---

## 8. Conclusion

In this practical guide, you learned how to deploy DNSSEC on CentOS 9 step-by-step:

- **Key Generation:** We generated both ZSK and KSK keys.
- **Zone Signing:** We signed an existing zone file, producing a file with RRSIG and DNSKEY records.
- **BIND Configuration:** We updated the named configuration to use the signed zone and enabled DNSSEC options.
- **Testing and Verification:** We used tools like dig and dnssec-verify to validate our DNSSEC deployment.
- **Key Management Overview:** We also covered best practices for key management and rollovers in a DNSSEC-enabled environment.

By following these steps, you secure your DNS infrastructure from spoofing and man-in-the-middle attacks, thus ensuring the integrity and authenticity of DNS responses.

*This completes the practical guide for DNSSEC on CentOS 9. Enjoy your secure DNS configuration!*
