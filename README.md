# rdp-sec-check

Perl script to enumerate security settings of an RDP Service (AKA Terminal Services)

## Key features

- Support for targets file
- Support for saving the tool output to a specified logfile
- Control over the connection and responses timeouts
- Control over the number of retries when timeouts occurs

## Overview

rdp-sec-check is a Perl script to enumerate the different security settings of an remote destktop service (AKA Terminal Services).

It does not require authentication, only network connectivity to TCP port 3389.

It can determine many (though not quite all) of the security settings from the RDP-Tcp Properties | General tab:

- Check which security layers are supported by the service: Standard RDP Security, TLSv1.0, CredSSP
- For Standard RDP Security it detects the level of encryption supported: 40-bit, 56-bit, 128-bit, FIPS

The following potential security issues are flagged if present:

- The service supports Standard RDP Security – rhis is known to be vulnerable to an active “Man-In-The-Middle” attack
- The service supports weak encryption (40-bit or 56-bit)
- The service does not mandate Network Level Authentication (NLA) - NLA can help to prevent certain types of Denial of Service attack
- The service supports FIPS encryption but doesn’t mandate it – may only be interesting for jurisdictions where FIPS is required

## Requirements

rdp-sec-check is a simple Perl script that requires one module from CPAN. Run `cpan` as root then install the Encoding::BER module:

```
# cpan
cpan[1]> install Encoding::BER
```

## Examples

### Example output 1: An old Windows 2000 RDP service

```
$ rdp-sec-check.pl 10.0.0.94
Starting rdp-sec-check v0.8-beta ( https://labs.portcullis.co.uk/application/rdp-sec-check/ ) at Mon Jul  9 13:34:38 2012
 
Target:    10.0.0.94
IP:        10.0.0.94
Port:      3389
 
[+] Checking supported protocols
 
[-] Checking if RDP Security (PROTOCOL_RDP) is supported...Negotiation ignored - old Windows 2000/XP/2003 system?
[-] Checking if TLS Security (PROTOCOL_SSL) is supported...Negotiation ignored - old Windows 2000/XP/2003 system?
[-] Checking if CredSSP Security (PROTOCOL_HYBRID) is supported [uses NLA]...Negotiation ignored - old Windows 2000/XP/2003 system??
 
[+] Checking RDP Security Layer
 
[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_NONE...Not supported
[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_40BIT...Supported.  Server encryption level: ENCRYPTION_LEVEL_CLIENT_COMPATIBLE
[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_128BIT...Not supported
[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_56BIT...Supported.  Server encryption level: ENCRYPTION_LEVEL_CLIENT_COMPATIBLE
[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_FIPS...Not supported
 
[+] Summary of protocol support
 
[-] 10.0.0.94:3389 supports PROTOCOL_RDP   : TRUE
[-] 10.0.0.94:3389 supports PROTOCOL_HYBRID: FALSE
[-] 10.0.0.94:3389 supports PROTOCOL_SSL   : FALSE
 
[+] Summary of RDP encryption support
 
[-] 10.0.0.94:3389 has encryption level: ENCRYPTION_LEVEL_CLIENT_COMPATIBLE
[-] 10.0.0.94:3389 supports ENCRYPTION_METHOD_NONE   : FALSE
[-] 10.0.0.94:3389 supports ENCRYPTION_METHOD_40BIT  : TRUE
[-] 10.0.0.94:3389 supports ENCRYPTION_METHOD_128BIT : FALSE
[-] 10.0.0.94:3389 supports ENCRYPTION_METHOD_56BIT  : TRUE
[-] 10.0.0.94:3389 supports ENCRYPTION_METHOD_FIPS   : FALSE
 
[+] Summary of security issues
 
[-] 10.0.0.94:3389 has issue NLA_NOT_SUPPORTED_DOS
[-] 10.0.0.94:3389 has issue ONLY_RDP_SUPPORTED_MITM
[-] 10.0.0.94:3389 has issue WEAK_RDP_ENCRYPTION_SUPPORTED
 
rdp-sec-check v0.8-beta completed at Mon Jul  9 13:34:39 2012
```

### Example output 2: A Windows 2003 SP0 RDP service

```
$ rdp-sec-check.pl 10.0.0.93
Starting rdp-sec-check v0.8-beta ( https://labs.portcullis.co.uk/application/rdp-sec-check/ ) at Mon Jul  9 13:35:34 2012
 
Target:    10.0.0.93
IP:        10.0.0.93
Port:      3389
 
[+] Checking supported protocols
 
[-] Checking if RDP Security (PROTOCOL_RDP) is supported...Negotiation ignored - old Windows 2000/XP/2003 system?
[-] Checking if TLS Security (PROTOCOL_SSL) is supported...Negotiation ignored - old Windows 2000/XP/2003 system?
[-] Checking if CredSSP Security (PROTOCOL_HYBRID) is supported [uses NLA]...Negotiation ignored - old Windows 2000/XP/2003 system??
 
[+] Checking RDP Security Layer
 
[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_NONE...Not supported
[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_40BIT...Supported.  Server encryption level: ENCRYPTION_LEVEL_CLIENT_COMPATIBLE
[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_128BIT...Supported.  Server encryption level: ENCRYPTION_LEVEL_CLIENT_COMPATIBLE
[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_56BIT...Supported.  Server encryption level: ENCRYPTION_LEVEL_CLIENT_COMPATIBLE
[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_FIPS...Supported.  Server encryption level: ENCRYPTION_LEVEL_CLIENT_COMPATIBLE
 
[+] Summary of protocol support
 
[-] 10.0.0.93:3389 supports PROTOCOL_RDP   : TRUE
[-] 10.0.0.93:3389 supports PROTOCOL_HYBRID: FALSE
[-] 10.0.0.93:3389 supports PROTOCOL_SSL   : FALSE
 
[+] Summary of RDP encryption support
 
[-] 10.0.0.93:3389 has encryption level: ENCRYPTION_LEVEL_CLIENT_COMPATIBLE
[-] 10.0.0.93:3389 supports ENCRYPTION_METHOD_NONE   : FALSE
[-] 10.0.0.93:3389 supports ENCRYPTION_METHOD_40BIT  : TRUE
[-] 10.0.0.93:3389 supports ENCRYPTION_METHOD_128BIT : TRUE
[-] 10.0.0.93:3389 supports ENCRYPTION_METHOD_56BIT  : TRUE
[-] 10.0.0.93:3389 supports ENCRYPTION_METHOD_FIPS   : TRUE
 
[+] Summary of security issues
 
[-] 10.0.0.93:3389 has issue NLA_NOT_SUPPORTED_DOS
[-] 10.0.0.93:3389 has issue FIPS_SUPPORTED_BUT_NOT_MANDATED
[-] 10.0.0.93:3389 has issue ONLY_RDP_SUPPORTED_MITM
[-] 10.0.0.93:3389 has issue WEAK_RDP_ENCRYPTION_SUPPORTED
```

### Example output 3: A typical Windows 2003 RDP service

```
$ rdp-sec-check.pl 10.0.0.111
Starting rdp-sec-check v0.8-beta ( https://labs.portcullis.co.uk/application/rdp-sec-check/ ) at Mon Jul  9 13:36:56 2012
 
Target:    10.0.0.111
IP:        10.0.0.111
Port:      3389
 
[+] Checking supported protocols
 
[-] Checking if RDP Security (PROTOCOL_RDP) is supported...Supported
[-] Checking if TLS Security (PROTOCOL_SSL) is supported...Not supported - SSL_NOT_ALLOWED_BY_SERVER
[-] Checking if CredSSP Security (PROTOCOL_HYBRID) is supported [uses NLA]...Not supported - SSL_NOT_ALLOWED_BY_SERVER
 
[+] Checking RDP Security Layer
 
[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_NONE...Not supported
[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_40BIT...Supported.  Server encryption level: ENCRYPTION_LEVEL_CLIENT_COMPATIBLE
[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_128BIT...Supported.  Server encryption level: ENCRYPTION_LEVEL_CLIENT_COMPATIBLE
[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_56BIT...Supported.  Server encryption level: ENCRYPTION_LEVEL_CLIENT_COMPATIBLE
[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_FIPS...Supported.  Server encryption level: ENCRYPTION_LEVEL_CLIENT_COMPATIBLE
 
[+] Summary of protocol support
 
[-] 10.0.0.111:3389 supports PROTOCOL_RDP   : TRUE
[-] 10.0.0.111:3389 supports PROTOCOL_HYBRID: FALSE
[-] 10.0.0.111:3389 supports PROTOCOL_SSL   : FALSE
 
[+] Summary of RDP encryption support
 
[-] 10.0.0.111:3389 has encryption level: ENCRYPTION_LEVEL_CLIENT_COMPATIBLE
[-] 10.0.0.111:3389 supports ENCRYPTION_METHOD_NONE   : FALSE
[-] 10.0.0.111:3389 supports ENCRYPTION_METHOD_40BIT  : TRUE
[-] 10.0.0.111:3389 supports ENCRYPTION_METHOD_128BIT : TRUE
[-] 10.0.0.111:3389 supports ENCRYPTION_METHOD_56BIT  : TRUE
[-] 10.0.0.111:3389 supports ENCRYPTION_METHOD_FIPS   : TRUE
 
[+] Summary of security issues
 
[-] 10.0.0.111:3389 has issue NLA_NOT_SUPPORTED_DOS
[-] 10.0.0.111:3389 has issue FIPS_SUPPORTED_BUT_NOT_MANDATED
[-] 10.0.0.111:3389 has issue ONLY_RDP_SUPPORTED_MITM
[-] 10.0.0.111:3389 has issue WEAK_RDP_ENCRYPTION_SUPPORTED
 
rdp-sec-check v0.8-beta completed at Mon Jul  9 13:36:56 2012
```

### Example output 4: A well configured Windows 2008 RDP service

```
$ rdp-sec-check.pl 10.0.0.21
Starting rdp-sec-check v0.8-beta ( https://labs.portcullis.co.uk/application/rdp-sec-check/ ) at Mon Jul  9 13:32:30 2012
 
Target:    10.0.0.21
IP:        10.0.0.21
Port:      3389
 
[+] Checking supported protocols
 
[-] Checking if RDP Security (PROTOCOL_RDP) is supported...Not supported - HYBRID_REQUIRED_BY_SERVER
[-] Checking if TLS Security (PROTOCOL_SSL) is supported...Not supported - HYBRID_REQUIRED_BY_SERVER
[-] Checking if CredSSP Security (PROTOCOL_HYBRID) is supported [uses NLA]...Supported
 
[+] Checking RDP Security Layer
 
[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_NONE...Not supported
[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_40BIT...Not supported
[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_128BIT...Not supported
[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_56BIT...Not supported
[-] Checking RDP Security Layer with encryption ENCRYPTION_METHOD_FIPS...Not supported
 
[+] Summary of protocol support
 
[-] 10.0.0.21:3389 supports PROTOCOL_RDP   : FALSE
[-] 10.0.0.21:3389 supports PROTOCOL_HYBRID: TRUE
[-] 10.0.0.21:3389 supports PROTOCOL_SSL   : FALSE
 
[+] Summary of RDP encryption support
 
[-] 10.0.0.21:3389 supports ENCRYPTION_METHOD_NONE   : FALSE
[-] 10.0.0.21:3389 supports ENCRYPTION_METHOD_40BIT  : FALSE
[-] 10.0.0.21:3389 supports ENCRYPTION_METHOD_128BIT : FALSE
[-] 10.0.0.21:3389 supports ENCRYPTION_METHOD_56BIT  : FALSE
[-] 10.0.0.21:3389 supports ENCRYPTION_METHOD_FIPS   : FALSE
 
[+] Summary of security issues
 
rdp-sec-check v0.8-beta completed at Mon Jul  9 13:32:31 2012
```

## Note

See https://labs.portcullis.co.uk/tools/rdp-sec-check/
