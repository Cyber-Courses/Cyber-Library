---
title: "Windows: attacking the Windows host platform"
description: "Offensive techniques against a Windows host: local privilege escalation through service, registry, and token misconfiguration, credential access from LSASS, SAM, and the DPAPI stores, UAC and integrity-level bypass, persistence through Windows startup and scheduling, and kernel and driver exploitation."
keywords:
  - Windows privilege escalation
  - LSASS
  - DPAPI
  - UAC bypass
  - token impersonation
---

# Windows

Windows is the dominant enterprise endpoint and server platform, and its local attack surface is the richest of the three operating systems. Once code is running as an ordinary user, the work is to escalate to `SYSTEM`, harvest the credentials and secrets the machine caches, and establish footholds that survive a reboot, all before, or instead of, reaching the surrounding Active Directory domain.

## The local surface

- **Privilege escalation**: unquoted and weak-permission service paths, service and task misconfiguration, abusable privileges (`SeImpersonate`, `SeBackup`, `SeDebug` and the potato-family token attacks), DLL hijacking and search-order abuse, Always-Install-Elevated, and vulnerable third-party and signed drivers.
- **Credential access**: dumping LSASS for plaintext and hashes, the SAM and SECURITY hives, cached domain credentials, DPAPI-protected secrets (browser, credential manager), and tokens for impersonation.
- **Integrity and UAC**: moving between integrity levels and the many user-account-control bypasses that elevate without a prompt.
- **Persistence**: run keys, services, scheduled tasks, WMI event subscriptions, COM hijacking, and startup folders.
- **Kernel**: local kernel and driver exploitation for the final step to `SYSTEM` or to defeat protections.

## Seams

Active Directory (domain authentication, Kerberos, NTLM, AD CS) is a separate area under [Directory](../../applications/server/directory/index.md); it is reached from a Windows foothold but attacked as a directory. This area is the standalone Windows host.

## References

- [MITRE ATT&CK: Windows privilege escalation](https://attack.mitre.org/tactics/TA0004/)
- [The Hacker Recipes: Windows](https://www.thehacker.recipes/)
