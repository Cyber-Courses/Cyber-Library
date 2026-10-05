---
title: "Version detection: which SNMP versions an agent accepts"
description: "An agent may answer SNMPv1, v2c, v3, or several at once, and the version decides the attack. Probing each version and observing which return data, and whether v3 is offered, reveals the exposure: a device still accepting v1/v2c is attackable by community string even if v3 is also configured, which is the common downgrade opportunity."
keywords:
  - snmp version detection
  - v1 v2c v3
  - downgrade
  - nmap snmp
  - probe
---

# Version detection

Determining which SNMP versions an agent accepts is the prerequisite step, because the versions have different attacks and agents frequently accept more than one. Probing with each version and seeing which return data tells you the exposure. A device that answers v1 or v2c is attackable through the cleartext community string regardless of whether v3 is also configured, which is the common and valuable finding: administrators enable v3 but leave v1/v2c on for legacy tooling, so the weak path remains. Conversely, a device that answers only v3 requires the v3-specific attacks.

```bash
# probe each version against a known OID
snmpget -v1  -c public <target> 1.3.6.1.2.1.1.1.0 2>/dev/null && echo "v1 ok"
snmpget -v2c -c public <target> 1.3.6.1.2.1.1.1.0 2>/dev/null && echo "v2c ok"
# v3 presence: engine discovery returns an engine ID even without credentials
snmpget -v3 -l noAuthNoPriv -u probe <target> 1.3.6.1.2.1.1.1.0 2>&1 | grep -i engineID
# nmap confirms the agent and attempts version/strings
nmap -sU -p161 -sV --script snmp-info <target>
```

## Exploitation notes

- The key finding is a device that still accepts v1/v2c: it is the weak path, and its presence alongside v3 is a downgrade opportunity, use the community-string attacks and ignore v3.
- A device answering only v3 forces the v3 attacks (username enumeration, offline cracking, downgrade attempt); confirm by the absence of any v1/v2c response and the presence of an engine ID on v3 discovery.
- Agents are UDP and silent to wrong credentials, so distinguish "wrong string" from "version not supported" by testing a known-good string per version where you have one.
- The detected version routes to [SNMPv1 and v2c cleartext](snmpv1-and-v2c-cleartext.md) or [SNMPv3 attacks](snmpv3-attacks.md).

## References

- [nmap snmp-info](https://nmap.org/nsedoc/scripts/snmp-info.html)
- [HackTricks: SNMP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
