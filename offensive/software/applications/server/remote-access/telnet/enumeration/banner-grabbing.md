---
title: "Banner grabbing: Telnet service and version disclosure"
description: "A Telnet service may send a banner or login prompt containing product and operating-system clues. Because banners can be generic, customized, or absent, corroborate the inferred model and version before applying credential policy or vulnerability research."
keywords:
  - telnet banner
  - version disclosure
  - device identification
  - nc 23
  - fingerprint
---

# Banner grabbing

Connecting to Telnet may yield negotiation bytes, a banner, or a login prompt. The response can contain useful clues such as an OS family, vendor branding, a device model, or a software version, but administrators can customize it and many services disclose little. Use it as one fingerprinting input and corroborate the exact product and version before applying a factory-credential policy or version-specific advisory.

```bash
nc <target> 23                                 # read the banner/login header
echo | nc -w3 <target> 23 | head
nmap -p23 -sV <target>                         # version detection
# for an authorized target list (preserves whitespace and avoids word splitting)
while IFS= read -r host; do echo "== $host =="; echo | nc -w2 "$host" 23 | head -5; done < hosts.txt
```

## Exploitation notes

- A sufficiently specific banner can support a [default-credential](../authentication/default-credentials.md) lookup, but a generic prompt is not enough to select a vendor or model.
- Firmware or version strings can guide a review of an embedded `telnetd` advisory, but vulnerability status also depends on vendor patches, backports, build options, and reachability; see [Memory corruption](../memory-corruption/index.md).
- Cisco and other network-gear banners reveal the OS family and sometimes the model, guiding both credential and exploit choices.
- Banner collection is normally pre-authentication, but it still creates a network connection that may be logged, rate-limited, or blocked. Keep the scope and rate appropriate for the assessment.

## References

- [HackTricks: Telnet banner](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
- [RFC 854 (Telnet)](https://datatracker.ietf.org/doc/html/rfc854)
