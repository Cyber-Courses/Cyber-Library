---
title: "SNMPv1 and v2c cleartext: capturing the community string"
description: "SNMPv1 and v2c send the community string in cleartext inside every request and provide no encryption, so a positioned attacker captures the string by sniffing UDP 161 traffic and reads all SNMP data passively. The captured string is then reused directly, and the exposure applies even when a stronger v3 is also configured but v1/v2c remains enabled."
keywords:
  - snmpv1
  - snmpv2c
  - cleartext
  - sniffing
  - community capture
---

# SNMPv1 and v2c cleartext

SNMPv1 and v2c offer no confidentiality: the community string is embedded in cleartext in every request packet, and the data in responses is unencrypted. A positioned attacker therefore defeats these versions passively, by capturing SNMP traffic (a management station polling a device, or a device sending traps) and reading the community string straight out of the packets, after which they reuse it. Even the data is exposed to a sniffer without needing the string at all. This matters especially where administrators enabled SNMPv3 but left v1/v2c enabled for legacy monitoring: the cleartext path undermines the stronger one, because the same device data (and often the same effective access) is reachable over the weak version.

```bash
# capture SNMP traffic and read the community string from the packets
tcpdump -i eth0 -A udp port 161 or udp port 162          # requests (161) and traps (162)
# Wireshark: the "snmp.community" field shows the string in each v1/v2c PDU
tshark -i eth0 -Y 'snmp.community' -T fields -e snmp.community -e ip.dst | sort -u
# then reuse the captured string
snmpbulkwalk -v2c -c <captured> <target>
```

## Exploitation notes

- The community string is in every v1/v2c packet in cleartext, so a single captured request (from a polling management station or any client) yields it; no cracking is involved.
- Capture needs an on-path or tap position where SNMP flows (near a monitoring server, a SPAN port, or a shared segment); management traffic to many devices often converges there, exposing many strings at once.
- The presence of v1/v2c alongside v3 is the exploitable gap: attack the weak version and skip v3 entirely.
- A captured string drives [enumeration](../enumeration/index.md) and, if read-write, [write access](../write-access.md); it is also frequently reused across the estate.

## References

- [RFC 1157 (SNMPv1)](https://datatracker.ietf.org/doc/html/rfc1157)
- [HackTricks: SNMP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
