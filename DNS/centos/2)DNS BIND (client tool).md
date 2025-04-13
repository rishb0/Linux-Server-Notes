# DNS-Server

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
