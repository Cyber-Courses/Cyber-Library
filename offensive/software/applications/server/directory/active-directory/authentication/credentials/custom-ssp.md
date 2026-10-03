---
title: "Custom SSP: logging cleartext credentials on the host"
description: "Registering a malicious Security Support Provider on a domain controller or server so the authentication subsystem records every logon's cleartext password, including service and machine account passwords, to a local file."
keywords:
  - custom SSP
  - mimilib
  - memssp
  - security packages
  - domain persistence
---

# Custom SSP

Windows authentication is pluggable: **Security Support Providers (SSPs)** are DLLs loaded into LSASS that handle authentication exchanges. Registering a **malicious SSP** makes LSASS hand every authentication, interactive logons, service starts, machine-account activity, to your code, which records the **cleartext** password. On a domain controller that is a steady stream of credentials, including service and computer account passwords, captured as users and services authenticate.

## Installing it

Two common forms, both needing admin/SYSTEM on the target:

```text
# Persistent: drop mimilib.dll and register it as a Security Package (survives reboot)
#   copy mimilib.dll to C:\Windows\System32\
#   add "mimilib" to HKLM\System\CurrentControlSet\Control\Lsa\Security Packages (REG_MULTI_SZ)

# In-memory: patch LSASS to add a logging SSP without touching disk (not reboot-persistent)
mimikatz: privilege::debug ; misc::memssp
```

Captured credentials are written in cleartext to a local log:

```text
# Both mimilib and memssp log to:
C:\Windows\System32\kiwissp.log
```

## Why it is valuable

- It yields **plaintext** passwords, not hashes, so no cracking is needed, and it catches credentials that are never in a dumpable hash form at rest.
- On a **DC**, it harvests high-value service and machine account passwords continuously as the domain operates.
- The **registry (`Security Packages`)** form reloads on reboot, making it durable; the `memssp` form is stealthier (no disk artifact) but clears on reboot.

## Exploitation notes

- Requires **LSA Protection (RunAsPPL)** to be off or bypassed, since the SSP loads into LSASS; a signed/PPL-enforced LSASS blocks an unsigned package.
- It is a harvesting backdoor, not instant access: value accrues as logons happen, so leave it and collect.
- Reading `kiwissp.log` needs local access to the host, so pair it with a re-entry method ([DSRM](dsrm.md), a [golden ticket](../kerberos/forged-tickets.md)).

## Tools

- **Mimikatz** (`misc::memssp`, and the `mimilib.dll` SSP): in-memory and on-disk logging SSPs.

## References

- [MITRE ATT&CK T1547.005: Security Support Provider](https://attack.mitre.org/techniques/T1547/005/)
- [pentestlab: persistence, Security Support Provider](https://pentestlab.blog/2019/10/21/persistence-security-support-provider/)
- [bufu-sec wiki: custom SSPs](https://wiki.bufu-sec.com/active-directory/persistence/custom_ssps)
