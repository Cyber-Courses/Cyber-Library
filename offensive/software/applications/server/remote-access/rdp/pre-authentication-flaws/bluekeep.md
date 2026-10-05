---
title: "BlueKeep: unauthenticated RDP remote code execution"
description: "BlueKeep is a pre-authentication use-after-free in the RDP handling of virtual channels, reachable before login on older Windows without Network Level Authentication. An unauthenticated attacker triggers the flaw via a crafted channel request to execute code as SYSTEM, and the bug is wormable, which drove emergency patching of end-of-life Windows."
keywords:
  - bluekeep
  - use-after-free
  - virtual channel
  - mst120
  - wormable
---

# BlueKeep

BlueKeep is a pre-authentication use-after-free in the RDP server's handling of virtual channels, specifically the internal binding of a static channel that let an attacker free and then reuse a kernel object before authentication. It affects older Windows (the Windows 7 / Server 2008 R2 era and earlier, including end-of-life Windows XP/2003) and is reachable only when Network Level Authentication is not enforced, because it lives in the pre-session code. An unauthenticated attacker triggers it with crafted channel setup and heap grooming to gain code execution as SYSTEM. Its wormability prompted Microsoft to patch even out-of-support Windows.

```bash
# identify exposure: affected build + NLA off
nmap -p3389 --script rdp-ntlm-info,rdp-enum-encryption <target>
# scanners/exploit modules exist (use the one matching the exact target build)
#   the exploit binds the vulnerable channel, grooms the kernel pool, and triggers
#   the UAF to execute a payload as SYSTEM; kernel-pool layout makes it build-sensitive.
```

## Exploitation notes

- Two preconditions: an affected Windows build (Windows 7/Server 2008 R2 and earlier) and NLA not required; `rdp-ntlm-info` gives the build and `rdp-enum-encryption` the NLA status.
- The flaw is in the pre-authentication channel handling, so NLA (CredSSP first) blocks reaching it; enforcing NLA was the stopgap mitigation before patching.
- Reliable exploitation requires grooming the kernel pool for the specific target, so it is build- and configuration-sensitive and can crash the host if the layout is wrong; this is the standard risk of the kernel-UAF class.
- Success is SYSTEM with no credentials, and the bug's wormability means a single exploited host can be used to spread; the follow-on newer-build class is [DejaBlue](dejablue.md).

## References

- [Microsoft advisory: BlueKeep](https://msrc.microsoft.com/update-guide/)
- [NCC/ZDI BlueKeep analyses](https://www.zerodayinitiative.com/blog)
