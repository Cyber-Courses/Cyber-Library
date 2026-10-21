---
title: "Encryption-key overflow: CVE-2011-4862 in affected telnetd implementations"
description: "CVE-2011-4862 was a pre-authentication buffer overflow in the Telnet encryption-option code used by specific FreeBSD and related telnetd implementations. An overlong encryption key could enable code execution with the daemon's privileges; encryption support alone does not establish vulnerability."
keywords:
  - buffer overflow
  - telnetd
  - CVE-2011-4862
  - encryption option
  - rce
---

# Encryption-key overflow

CVE-2011-4862 was a buffer overflow in `libtelnet/encrypt.c` used by specific BSD-derived Telnet daemons. The affected code copied a client-supplied encryption key into a fixed-size buffer without validating its length. Because this option was processed before authentication, a remote unauthenticated attacker could execute code with the privileges of the daemon, which was commonly root. The advisory covered named versions of FreeBSD and related Heimdal, MIT Kerberos applications, and GNU inetutils code; it was not a generic flaw in every Telnet server or embedded implementation.

```bash
# identify the service and whether it supports the relevant encryption option
nmap -p23 -sV --script telnet-encryption <target>
# support is only a lead: confirm an affected product/version against the advisory
```

## Exploitation notes

- The vulnerable encryption-key handling runs before authentication; successful exploitation inherits the daemon's privileges.
- `telnet-encryption` detects option support only. Nmap explicitly states that the script does not test for CVE-2011-4862.
- Confirm the exact implementation and version through inventory, package data, or vendor evidence; a banner may be insufficient.
- Do not extend this CVE to BusyBox or vendor forks without evidence that they contain the affected code.

## References

- [FreeBSD-SA-11:08.telnetd](https://www.freebsd.org/security/advisories/FreeBSD-SA-11:08.telnetd.asc)
- [CVE-2011-4862](https://nvd.nist.gov/vuln/detail/CVE-2011-4862)
- [Nmap telnet-encryption](https://nmap.org/nsedoc/scripts/telnet-encryption.html)
