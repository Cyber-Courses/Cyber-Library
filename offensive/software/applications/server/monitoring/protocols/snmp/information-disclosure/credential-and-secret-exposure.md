---
title: "Credential and secret exposure: keys and credentials in SNMP OIDs"
description: "SNMP agents expose secrets in both standard and vendor OIDs: other community strings and SNMPv3 user data, wireless (WEP/WPA) keys and VPN pre-shared keys on network gear, and credentials or tokens that applications place in custom enterprise OIDs. A read walk frequently recovers material that unlocks the device and others, beyond the device's own compromise."
keywords:
  - snmp credentials
  - wep wpa keys
  - vpn psk
  - custom oid
  - secret exposure
---

# Credential and secret exposure

Beyond topology and host data, SNMP frequently exposes outright secrets. Network devices store sensitive material in their enterprise MIBs that is readable over SNMP: wireless access points expose WEP/WPA keys and SSIDs, VPN endpoints expose pre-shared keys and configuration, and many devices expose their other, possibly stronger, SNMP community strings and SNMPv3 user settings, so a weak read string can leak the better credentials. Vendor and application agents also place data in custom enterprise OIDs (`1.3.6.1.4.1.<vendor>`) that developers did not consider secret, sometimes including credentials, tokens, or internal URLs. Walking the full tree, especially the enterprise subtrees, with attention to string values is how these are found.

```bash
# walk the vendor enterprise subtree and grep for secret-looking values
snmpbulkwalk -v2c -c public -Oqv <target> 1.3.6.1.4.1 \
  | grep -iE 'pass|pwd|secret|key|psk|wpa|wep|community|token|ssid' | head
# wireless gear: SSIDs and keys live in the vendor WLAN MIB (subtree is vendor-specific)
# the config-copy route on Cisco yields all device credentials at once (see Cisco page)
```

## Exploitation notes

- The vendor enterprise subtree (`1.3.6.1.4.1`) is where product-specific secrets live; dump it and pattern-match string values for credential-like content, since developers often expose data there without treating it as sensitive.
- A weak read string that exposes the device's other community strings or SNMPv3 users is a privilege step: use the leaked stronger string/credentials for write access or for devices the weak string does not cover.
- Wireless and VPN keys recovered here are reusable to join the network or decrypt traffic; device management credentials pivot to the device's other interfaces (SSH/HTTP).
- On Cisco, the complete credential set comes from the config itself; see [Cisco configuration exfiltration](cisco-configuration-exfiltration.md), which this complements for non-Cisco and application agents.

## References

- [HackTricks: SNMP secrets](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
- [Enterprise OID registry (IANA PEN)](https://www.iana.org/assignments/enterprise-numbers/)
