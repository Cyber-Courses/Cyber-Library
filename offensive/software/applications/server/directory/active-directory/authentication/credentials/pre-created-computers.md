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

When an administrator pre-stages a computer object with the **"assign this computer account as a pre-Windows 2000 computer"** option, Active Directory sets its password to a **predictable default: the computer name in lowercase**, first 14 characters, without the trailing `$`. (Objects created without that option get a random password, so this applies specifically to the pre-Windows 2000 case.) If such an object has never authenticated, the default usually still stands, which hands you a working **computer-account credential** for free.

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

The guessed password authenticates as the computer. Reset it to a value you control, using the RPC or kpasswd transport (the `smb-samr` default fails for a never-logged-on machine account):

```bash
# -protocol rpc-samr (or kpasswd) changes the initial password; the smb-samr default
# fails on an unused computer account with STATUS_NOLOGON_WORKSTATION_TRUST_ACCOUNT
changepasswd.py -protocol rpc-samr 'example.local/WKSTN01$:wkstn01@<dc>' -newpass 'Password123!'
```

- The result is an attacker-controlled, SPN-bearing **computer identity**. That is the account you nominate as the delegated principal in a [resource-based constrained delegation](../kerberos/delegation/resource-based-constrained.md) chain, but completing it still needs a separate write over the **target's** `msDS-AllowedToActOnBehalfOfOtherIdentity`; the pre-created account supplies the controlled principal, not the write over the target.
- On its own it is a plain authenticated foothold for [LDAP enumeration](ldap-enumeration.md) and [roasting](../kerberos/roasting.md) when you started with nothing.

## Exploitation notes

- This is a **legacy default** that survives for years: the pre-Windows 2000 compatibility option predates modern joins, but admins still pre-stage machines and leave the predictable password in place.
- `logonCount=0` with the `PASSWD_NOTREQD` flag is the strong signal; a machine that has logged on will have rotated its password.
- The owned computer account rarely has special rights by itself, so the value is as a **controlled principal for a delegation chain** (given a separate target-write) and as an authenticated foothold, not the account's own access.

## Tools

- **pre2k** (garrettfoster13): discover and authenticate to pre-created computers.
- **Impacket** (`changepasswd.py`, `addcomputer.py`): reset the machine password and operate the account.

## References

- [The Hacker Recipes: pre-Windows 2000 computers](https://www.thehacker.recipes/ad/movement/builtins/pre-windows-2000-computers)
- [TrustedSec: diving into pre-created computer accounts](https://trustedsec.com/blog/diving-into-pre-created-computer-accounts)
- [pre2k (garrettfoster13)](https://github.com/garrettfoster13/pre2k)
