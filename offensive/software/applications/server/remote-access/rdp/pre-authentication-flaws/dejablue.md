---
title: "DejaBlue: pre-auth RDP RCE in newer Windows builds"
description: "DejaBlue is the collective name for a set of RDP pre-authentication and related vulnerabilities disclosed after BlueKeep that extended the wormable RCE risk to newer Windows versions. Like BlueKeep they are reachable before authentication where Network Level Authentication is not enforced, giving unauthenticated code execution against in-range builds."
keywords:
  - dejablue
  - rdp pre-auth
  - newer windows
  - wormable
  - rdp rce
---

# DejaBlue

DejaBlue is the collective label for a group of RDP vulnerabilities disclosed after BlueKeep that extended the pre-authentication, wormable RCE risk to newer Windows versions (the Windows 8/10 and Server 2012/2016/2019 range), which BlueKeep did not affect. They sit in the RDP protocol handling and, like BlueKeep, are reachable before authentication on hosts where Network Level Authentication is not enforced, so an unauthenticated attacker against an in-range, unpatched, NLA-off host gains code execution. The set closed the gap that left modern Windows exposed to the same class of attack.

```bash
# identify exposure: affected newer build + NLA off
nmap -p3389 --script rdp-ntlm-info,rdp-enum-encryption <target>   # build + NLA status
# exploitation matches the specific DejaBlue variant and build; the preconditions
# (unpatched in-range build, NLA not required) are the same as BlueKeep.
```

## Exploitation notes

- DejaBlue matters because it covers the modern Windows builds BlueKeep did not, so "newer OS" is not by itself safety; fingerprint the exact build and patch level.
- The preconditions mirror BlueKeep: an affected build and NLA not required; enforcing NLA blocks the pre-auth reach, and patching closes the bug.
- As with BlueKeep, these are memory-corruption flaws whose reliable exploitation is build-sensitive and can destabilise the host; match the exploit to the precise target.
- Where NLA is required or the host is patched, pivot to the [authentication](../authentication/index.md) and [exposure](../exposure/index.md) surfaces instead.

## References

- [Microsoft advisory: DejaBlue RDP vulnerabilities](https://msrc.microsoft.com/update-guide/)
- [HackTricks: RDP vulnerabilities](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
