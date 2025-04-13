# DNS-Server

# DNS Client Tools

DNS client tools are utilities used to query and troubleshoot DNS issues. They provide methods for verifying zone information, checking name resolution, and testing configurations.

## DNS Client Tools Commands

Below are the common commands used to manage and query DNS client tools:

### Check Installed BIND Packages
Use the following command to list installed BIND packages on Debian-based systems:
```
dpkg -l | grep bind
```

### Check Installed BIND Utilities
List installed `bind9-dnsutils` packages using the following command:
```
dpkg -l | grep bind9-dnsutils
```

### Install BIND Utilities
To install BIND utilities (such as dig, host, nslookup, etc.) using `apt`:
```
apt update
apt install bind9-dnsutils
```

### List Files Installed by bind9-dnsutils
To list all files installed by the `bind9-dnsutils` package:
```
dpkg -L bind9-dnsutils
```

### List Configuration Files for bind9-dnsutils
To list configuration files provided by the `bind9-dnsutils` package:
```
dpkg -L bind9-dnsutils | grep etc
```

### List Documentation Files for bind9-dnsutils
To list documentation files for the `bind9-dnsutils` package:
```
dpkg -L bind9-dnsutils | grep doc
```

### Common DNS Client Tools Installed with bind9-dnsutils

These are the commonly used DNS client tools included with `bind9-dnsutils`:
- **/usr/bin/delv** – DNS lookup and validation utility, providing extended security information.
- **/usr/bin/dig** – Query DNS servers for information about domains. It is versatile and commonly used for troubleshooting.
- **/usr/bin/host** – Simple utility for performing DNS lookups.
- **/usr/bin/nslookup** – A legacy tool to query DNS servers; while deprecated in some systems, it is still widely available for compatibility.
- **/usr/bin/nsupdate** – Utility for performing dynamic DNS updates, allowing automated changes to zone files.