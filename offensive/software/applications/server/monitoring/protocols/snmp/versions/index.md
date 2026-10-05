---
title: "Versions: SNMP version weaknesses"
description: "SNMP has three versions with very different security. v1 and v2c authenticate with a cleartext community string and no encryption, so they are sniffable and trivially abused. v3 adds user-based authentication and privacy, but it still leaks usernames before authentication and can be attacked offline or downgraded. Detecting the version decides the attack."
keywords:
  - snmp version
  - snmpv1
  - snmpv2c
  - snmpv3
  - version detection
---

# Versions

SNMP's three versions differ fundamentally in security, so identifying which an agent speaks decides the whole attack. SNMPv1 and v2c share the same weak model: the community string is the only credential, it is sent in cleartext, and nothing is encrypted, so the string is captured by sniffing and everything is exposed to anyone who has it. SNMPv3 is a different protocol with user-based security, optional authentication and privacy (encryption), which removes the cleartext-string weakness, but it still discloses usernames during the engine-discovery handshake and is subject to offline cracking of captured authentication and to downgrade where weaker versions remain enabled. Many agents answer multiple versions at once.

## Subtopics

- **[Version detection](version-detection.md)**: determining which versions an agent accepts.
- **[SNMPv1 and v2c cleartext](snmpv1-and-v2c-cleartext.md)**: the cleartext community-string weakness.
- **[SNMPv3 attacks](snmpv3-attacks.md)**: username enumeration, offline cracking, and downgrade.

## References

- [RFC 3414 (SNMPv3 USM)](https://datatracker.ietf.org/doc/html/rfc3414)
- [HackTricks: SNMP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
