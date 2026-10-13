---
title: "Null sessions: anonymous enumeration and RID cycling"
order: 20
description: "Enumerating a domain without any credentials by abusing anonymous SMB and LDAP access: null-session IPC$ connections, anonymous LDAP binds, and RID cycling through LSARPC/SAMR to recover the user list where anonymous access was never locked down."
keywords:
  - null session
  - anonymous enumeration
  - RID cycling
  - RestrictAnonymous
  - rpcclient
---

# Null sessions

Before any credential, it is worth asking what the domain gives away to **nobody at all**. A **null session** is an unauthenticated connection to a Windows host's `IPC$` share (empty username and password) that historically exposed users, groups, shares, and policy over SMB/RPC. Anonymous **LDAP** binds and anonymous **RID cycling** are the same idea against the directory. These are the oldest enumeration techniques in the Windows playbook, and they still pay off wherever the anonymous-access restrictions were never tightened.

## A long lockdown

Null sessions were wide open on NT 4.0 and Windows 2000, where `IPC$` anonymous access returned the full SAM. Microsoft progressively restricted them with `RestrictAnonymous`, `RestrictAnonymousSAM`, and `everyoneincludesanonymous`, so modern member servers usually refuse the juicy calls. But domain controllers still answer some anonymous LSARPC/SAMR queries by design, legacy and appliance systems often keep null sessions enabled, and anonymous LDAP binds reappear on misconfigured DCs, so the technique never fully died.

## Anonymous SMB and RID cycling

```bash
# Null-session connection and enumeration
rpcclient -U "" -N <target>
#  rpcclient $> enumdomusers ; querydispinfo ; lsaenumsid

# RID cycling: resolve SIDs to names to rebuild the user list without credentials
nxc smb <target> -u '' -p '' --rid-brute
enum4linux-ng -A <target>
```

## Anonymous LDAP

```bash
# Anonymous bind against the DC; where allowed, dump naming contexts and accounts
ldapsearch -x -H ldap://<dc> -b "DC=example,DC=local" "(objectClass=user)" sAMAccountName
```

## Exploitation notes

- The payoff is a **user list and domain policy with zero credentials**, which seeds [password spraying](password-spraying.md) and [AS-REP roasting](../kerberos/roasting.md).
- **RID cycling** works even when `enumdomusers` is blocked, because resolving SID-500, SID-1000, SID-1001 ... by hand still returns names where SAMR/LSARPC answer anonymously.
- Domain controllers frequently leak more anonymously than member servers, so always test the DC directly.
- A [null-session](user-and-group-enumeration.md) user list is often indistinguishable in value from an authenticated one for the next step, so this is a genuine pre-credential foothold, not just a curiosity.

## Tools

- **rpcclient** (`-U "" -N`): raw null-session RPC enumeration.
- **enum4linux-ng**: wraps the classic null-session and RID-cycling checks.
- **NetExec** (`-u '' -p '' --rid-brute`, `--users`, `--shares`): anonymous SMB enumeration.
- **ldapsearch** (`-x`): anonymous LDAP binds.

## References

- [enum4linux-ng (cddmp)](https://github.com/cddmp/enum4linux-ng)
- [HackTricks: SMB enumeration and null sessions](https://hacktricks.wiki/en/network-services-pentesting/pentesting-smb/index.html)
- [Microsoft: Network access, do not allow anonymous enumeration of SAM accounts](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/network-access-do-not-allow-anonymous-enumeration-of-sam-accounts)
