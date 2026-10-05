---
title: "Pre-authentication flaws: wormable RDP protocol RCE"
description: "RDP's pre-session protocol handling has produced wormable, unauthenticated remote code execution vulnerabilities, the BlueKeep and DejaBlue class, reachable before login on hosts without Network Level Authentication. Fingerprinting the Windows build and NLA status identifies hosts in the affected range, where exploitation gives SYSTEM with no credentials."
keywords:
  - rdp pre-auth rce
  - bluekeep
  - dejablue
  - wormable
  - nla
---

# Pre-authentication flaws

The most severe RDP vulnerabilities are pre-authentication: memory-corruption bugs in the protocol handling that runs before a user logs in, reachable by an unauthenticated client and giving code execution as SYSTEM. Because RDP is so widely exposed, these are wormable, capable of self-propagating across networks, which is why they drew urgent, out-of-cycle patches and comparisons to the SMB worm events. They live in the pre-session code, so Network Level Authentication (which forces authentication first) blocks reaching them; an NLA-off host of the right build is exposed. The two named classes are BlueKeep and DejaBlue.

```bash
# identify hosts in the affected build range with NLA off
nmap -p3389 --script rdp-ntlm-info,rdp-enum-encryption <target>   # build + NLA status
nmap -p3389 --script rdp-vuln-ms12-020 <target>                   # (older RDP DoS/vuln check)
```

## Subtopics

- **[BlueKeep](bluekeep.md)**: the pre-auth RCE in the pre-NLA RDP code path.
- **[DejaBlue](dejablue.md)**: the follow-on pre-auth RCE class affecting newer builds.

## References

- [Microsoft advisory: RDP pre-auth RCE](https://msrc.microsoft.com/update-guide/)
- [HackTricks: RDP vulnerabilities](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
