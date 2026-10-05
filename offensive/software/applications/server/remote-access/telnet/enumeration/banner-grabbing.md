---
title: "Banner grabbing: Telnet service and version disclosure"
description: "A Telnet server usually sends a banner on connection, often naming the operating system, device model, and software version, and network equipment and IoT devices are especially verbose. That banner identifies the target for default-credential lookup and for matching old telnetd builds to their memory-corruption vulnerabilities."
keywords:
  - telnet banner
  - version disclosure
  - device identification
  - nc 23
  - fingerprint
---

# Banner grabbing

Connecting to Telnet almost always yields a banner before or at the login prompt, and it is frequently rich: it names the OS (a Linux login banner, a Cisco IOS header, a BusyBox prompt), the device vendor and model for network gear and IoT, and sometimes the exact firmware or software version. This is the primary fingerprint for Telnet, because the device and version map directly to documented default credentials and to the specific `telnetd` implementation (and its known RCE bugs).

```bash
nc <target> 23                                 # read the banner/login header
echo | nc -w3 <target> 23 | head
nmap -p23 -sV <target>                         # version detection
# at scale
for h in $(cat hosts.txt); do echo "== $h =="; echo | nc -w2 $h 23 | head -5; done
```

## Exploitation notes

- The banner commonly identifies the device class (router/switch/camera/printer/IoT) and vendor, which is exactly what a [default-credential](../authentication/default-credentials.md) lookup needs.
- Firmware/version strings map an embedded `telnetd` (often BusyBox or a vendor build) to its known memory-corruption flaws, see [Memory corruption](../memory-corruption/index.md).
- Cisco and other network-gear banners reveal the OS family and sometimes the model, guiding both credential and exploit choices.
- Everything here is pre-auth and cleartext; it costs nothing and should precede any authentication attempt.

## References

- [HackTricks: Telnet banner](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
- [RFC 854 (Telnet)](https://datatracker.ietf.org/doc/html/rfc854)
