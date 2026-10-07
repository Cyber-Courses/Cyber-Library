---
title: "Targeted Kerberoasting: writing an SPN to roast a user"
order: 4
description: "Using WriteSPN (or GenericWrite) over an Active Directory user to set a temporary servicePrincipalName, request and crack its service ticket, then remove the SPN, turning a write over an account into its password offline."
keywords:
  - targeted kerberoasting
  - WriteSPN
  - servicePrincipalName
  - targetedKerberoast
  - GenericWrite
---

# Targeted Kerberoasting

Ordinary [Kerberoasting](../authentication/kerberos/roasting.md) targets accounts that already have a Service Principal Name. **Targeted** Kerberoasting manufactures that condition: with the right to write a user's **`servicePrincipalName`** (the `WriteSPN` edge, or any `GenericWrite`/`GenericAll`), you set an arbitrary SPN on the victim, request a service ticket for it, crack it offline, and remove the SPN. It converts a write over a user into that user's password without ever resetting it.

## The sequence

```bash
# Automated: set an SPN on every writable user, roast, then clean up
targetedKerberoast.py -v -d example.local -u user -p pass

# Manual with bloodyAD
bloodyAD --host <dc> -d example.local -u user -p pass set object 'victim' servicePrincipalName -v 'fake/svc'
GetUserSPNs.py -request-user victim example.local/user:pass -dc-ip <dc> -outputfile roast.hash
bloodyAD --host <dc> -d example.local -u user -p pass set object 'victim' servicePrincipalName  # clear
hashcat -m 13100 roast.hash wordlist.txt -r rules/best64.rule
```

`targetedKerberoast.py` does the whole loop (find writable users, add a temporary SPN, roast, restore) and is the usual tool.

## Why choose it

- It is **non-destructive**: unlike a [password reset](password-reset.md) it never changes the victim's password, so the account keeps working and the action is quiet.
- It only pays off if the account's password is **crackable**, so it suits human/service accounts with weak passwords, not machine accounts (whose random passwords will not crack).
- It is the natural move when a `GenericWrite` edge exists but PKINIT is unavailable for [shadow credentials](../authentication/kerberos/shadow-credentials.md); otherwise shadow credentials give direct access regardless of password strength.

## Exploitation notes

- Request **RC4** (etype 23, `-m 13100`) tickets where allowed; they crack far faster than AES.
- Always **remove** the SPN you added; leaving stray SPNs is both a mess and an indicator.
- Writing an SPN requires the value be unique in the forest and syntactically a valid SPN, but it need not resolve to anything, so `fake/svc` works.

## Tools

- **targetedKerberoast.py**: end-to-end write-roast-cleanup from Linux.
- **bloodyAD** / **PowerView** (`Set-DomainObject -Set @{serviceprincipalname=...}`): set/clear the SPN manually.
- **Impacket `GetUserSPNs.py`**, **hashcat -m 13100**: request and crack.

## References

- The Hacker Recipes: targeted Kerberoasting
- SpecterOps: BloodHound WriteSPN edge
