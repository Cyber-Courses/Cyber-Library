---
title: "Pre-created computer accounts: the default machine password"
description: "Taking over computer objects that were pre-staged in Active Directory but never joined, whose password defaults to the lowercase computer name, giving a free authenticated computer-account credential and a path into resource-based delegation or shadow credentials."
keywords:
  - pre-created computer
  - pre-Windows 2000
  - machine account password
  - pre2k
  - PASSWD_NOTREQD
---

# Pre-created computer accounts

When an administrator pre-stages a computer object (the "assign this computer account as a pre-Windows 2000 computer" option, or any created-but-never-joined machine), Active Directory sets its password to a **predictable default: the computer name in lowercase**, first 14 characters, without the trailing `$`. If that object has never authenticated, the default usually still stands, which hands you a working **computer-account credential** for free.

## Finding them

Candidates carry `PASSWD_NOTREQD` and `WORKSTATION_TRUST_ACCOUNT` in `userAccountControl` (the combined value `4128`) and a `logonCount` of `0`, meaning the account exists but has never logged on:

```bash
# pre2k: discover and test likely pre-created computers, authenticating with name-as-password
pre2k unauth -d example.local -dc-ip <dc> -inputfile computers.txt
pre2k auth -d example.local -dc-ip <dc> -u user -p pass
```

```text
# LDAP filter for the tell-tale flags and no prior logon
(&(objectClass=computer)(userAccountControl:1.2.840.113556.1.4.803:=4128)(logonCount=0))
```

## From a pre-created account to takeover

The guessed password authenticates as the computer. Reset it to a known value and the computer object is yours to use:

```bash
# Set a password you control, then use the computer for delegation or key credentials
changepasswd.py 'example.local/WKSTN01$:wkstn01@<dc>' -newpass 'Password123!'
```

- A controlled computer account is exactly what [resource-based constrained delegation](../kerberos/delegation/resource-based-constrained.md) and [shadow credentials](../kerberos/shadow-credentials.md) need, so a pre-created machine is a direct route to impersonating users on a target.
- It is also a plain authenticated foothold for [LDAP enumeration](../../reconnaissance/ldap-enumeration.md) and [roasting](../kerberos/roasting.md) when you started with nothing.

## Exploitation notes

- This is a **legacy default** that survives for years: the pre-Windows 2000 compatibility option predates modern joins, but admins still pre-stage machines and leave the predictable password in place.
- `logonCount=0` with the `PASSWD_NOTREQD` flag is the strong signal; a machine that has logged on will have rotated its password.
- The owned computer account rarely has special rights by itself, so the value is in using it as the **delegation or shadow-credential primitive**, not the account's own access.

## Tools

- **pre2k** (garrettfoster13): discover and authenticate to pre-created computers.
- **Impacket** (`changepasswd.py`, `addcomputer.py`): reset the machine password and operate the account.

## References

- [The Hacker Recipes: pre-Windows 2000 computers](https://www.thehacker.recipes/ad/movement/builtins/pre-windows-2000-computers)
- [TrustedSec: diving into pre-created computer accounts](https://trustedsec.com/blog/diving-into-pre-created-computer-accounts)
- [pre2k (garrettfoster13)](https://github.com/garrettfoster13/pre2k)
