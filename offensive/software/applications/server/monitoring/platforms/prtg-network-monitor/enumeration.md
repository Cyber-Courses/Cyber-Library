---
title: "Enumeration: fingerprinting PRTG"
description: "PRTG is identified by its web interface, login page, and version string, exposed on the core server ports. The version matters because the notification command-injection weakness and specific vulnerabilities are version-dependent, so reading the build from the login page or API directs the attack."
keywords:
  - prtg enumeration
  - version
  - login page
  - fingerprint
  - paessler
---

# Enumeration

Identifying PRTG and its version is quick and directs the attack. The web interface (the PRTG core server, default 8080/8443, often fronted on 80/443) has a recognizable login page and branding, and the version string appears in the page, resources, and API responses. The version matters because the notification-based code execution and the specific known vulnerabilities depend on the build and whether the relevant fixes are present. Reading the version from the login page or the API is the first step before authenticating.

```bash
# fingerprint PRTG and its version
curl -sk https://<target>/index.htm | grep -ioE 'PRTG[^<]*'
curl -sk 'https://<target>/api/status.json?id=0' | grep -i version   # API status (version)
curl -sk https://<target>/login.htm | grep -ioE 'Version [0-9.]+'
```

## Exploitation notes

- The version string from the login page or `api/status.json` maps to whether the notification command-injection fix and other patches are present, directing the [notification abuse](notification-command-abuse.md) and [known exploits](known-exploits.md).
- PRTG runs on Windows with the core service as Local System, so successful execution is SYSTEM; this is worth confirming the product for, as the impact is high.
- The interface may be on non-standard ports (8080/8443) even when 80/443 are fronted; scan for the core.
- Route to [authentication](authentication.md); the default `prtgadmin` is the common first win.

## References

- [PRTG API documentation](https://www.paessler.com/manuals/prtg/application_programming_interface_api_definition)
- [HackTricks: PRTG](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/prtg)
