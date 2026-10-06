---
title: "Skeleton Key: a master password patched into LSASS"
order: 10
description: "Patching the LSASS process on a domain controller so that every account authenticates with an attacker-chosen master password in addition to its real one, giving domain-wide access until the DC reboots."
keywords:
  - skeleton key
  - LSASS patch
  - misc skeleton
  - RC4 downgrade
  - domain persistence
---

# Skeleton Key

Skeleton Key patches the authentication path **in memory** on a domain controller so that, alongside each account's real password, a single **master password** of your choosing is also accepted. After it is applied, you can log in as **any** domain user with that one password while legitimate logons keep working, so nothing visibly breaks. It is fast domain-wide access, but it lives only in LSASS memory, so it does **not** survive a reboot.

## Applying it

With Domain Admin and the ability to run code as SYSTEM on a DC (or patch LSASS remotely), inject the key:

```text
# Mimikatz on the DC (or via remote code execution)
privilege::debug
misc::skeleton
```

After this, authenticate as any user with the master password (Mimikatz uses `mimikatz` as the master):

```bash
# From Linux, log in as any account with the skeleton password
nxc smb <dc> -u Administrator -p mimikatz
```

## Why it works, and its limits

- It patches the **RC4** path in LSASS, so it requires Kerberos RC4 to be available; environments forced to **AES-only** can defeat or complicate it, and re-patching for AES is needed.
- The patch is **in memory only**: a DC reboot clears it, so Skeleton Key is a *session-length* backdoor, re-applied as needed or paired with a reboot-surviving technique ([DSRM](dsrm.md), [custom SSP](custom-ssp.md), [golden ticket](../kerberos/forged-tickets.md)).
- Applying it reliably needs **LSA Protection (RunAsPPL) off** or a PPL bypass, and it touches a monitored process, so it is powerful but not subtle.

## Exploitation notes

- The appeal is **convenience**: one password logs you in as anyone, including accounts whose hashes you never dumped.
- Because real passwords still work, users and admins notice nothing, which buys dwell time until the next reboot.
- Treat it as tactical: use it for a window of broad access, and lay down a durable backdoor separately.

## Tools

- **Mimikatz** (`misc::skeleton`): the reference implementation.
- **Impacket / NetExec**: authenticate as any account with the master password once applied.

## References

- [MITRE ATT&CK T1556.001: Modify Authentication Process, Domain Controller Authentication](https://attack.mitre.org/techniques/T1556/001/)
- [pentestlab: Skeleton Key](https://pentestlab.blog/2018/04/10/skeleton-key/)
- [The Hacker Recipes: Skeleton Key](https://www.thehacker.recipes/ad/persistence/skeleton-key)
