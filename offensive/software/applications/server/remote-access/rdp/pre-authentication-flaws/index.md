---
title: "Pre-authentication flaws: wormable RDP protocol RCE"
description: "RDP's pre-session protocol handling has produced wormable, unauthenticated remote code execution vulnerabilities, including the BlueKeep and DejaBlue classes, reachable before login on affected, unpatched hosts where Network Level Authentication is not required."
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
# collect build information when CredSSP responds and enumerate accepted security layers
nmap -p3389 --script rdp-ntlm-info,rdp-enum-encryption <target>
# interpret the complete output: CredSSP support does not prove that NLA is required
nmap -p3389 --script rdp-vuln-ms12-020 <target>                   # (older RDP DoS/vuln check)
```

## Subtopics

- **[BlueKeep](bluekeep.md)**: the pre-auth RCE in the pre-NLA RDP code path.
- **[DejaBlue](dejablue.md)**: the follow-on pre-auth RCE class affecting newer builds.

## References

- [Microsoft advisory: RDP pre-auth RCE](https://msrc.microsoft.com/update-guide/)
- [HackTricks: RDP vulnerabilities](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
