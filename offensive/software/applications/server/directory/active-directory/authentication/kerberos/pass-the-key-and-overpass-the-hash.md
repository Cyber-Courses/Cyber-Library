---
title: "Pass-the-key and overpass-the-hash: turning a key into a ticket"
order: 2
description: "Requesting a Kerberos TGT directly from an account's secret key (NT hash or AES key) instead of its plaintext password, so a recovered hash becomes a full Kerberos identity (overpass-the-hash / pass-the-key)."
keywords:
  - overpass the hash
  - pass the key
  - AES key
  - TGT
  - PKINIT
---

# Pass-the-key and overpass-the-hash

Kerberos never needs the plaintext password: the AS exchange is keyed on the account's **secret key**, which is derived from the password (RC4 key = the NT hash; AES keys = PBKDF2 over the password). So if you hold the NT hash or an AES key, you can request a legitimate TGT and become that account in Kerberos. When the input is the NT hash this is called **overpass-the-hash**; when it is the AES key it is **pass-the-key**. They are the bridge from a recovered secret ([dumping](../credentials/lsass-dumping.md), [DCSync](../credentials/ntds-and-dcsync.md)) into full Kerberos movement.

## Requesting the TGT

```bash
# Overpass-the-hash: NT (RC4) hash -> TGT
getTGT.py -hashes :<nthash> example.local/user
# Pass-the-key: AES256 key -> TGT (stealthier where RC4 is monitored/disabled)
getTGT.py -aesKey <aes256key> example.local/user

export KRB5CCNAME=user.ccache
wmiexec.py -k -no-pass example.local/user@<host>        # use the ticket

# Rubeus, on-host
Rubeus.exe asktgt /user:user /rc4:<nthash> /ptt
Rubeus.exe asktgt /user:user /aes256:<aes256key> /ptt
```

The resulting TGT is indistinguishable from a normal logon's: this is a legitimate ticket, just obtained from the key instead of the password.

## Why choose this over pass-the-hash

[Pass-the-hash](../ntlm/pass-the-hash.md) reuses the NT hash over **NTLM**. Overpass-the-hash converts the same hash into **Kerberos**, which matters when:

- The target accepts only Kerberos (NTLM disabled or blocked).
- You want to move with tickets (pass-the-ticket, delegation abuse) rather than NTLM auth.
- You want to be quieter: a Kerberos logon blends with normal traffic better than NTLM exec in some environments.

## RC4 vs AES

- Supplying the **RC4** key (the NT hash) forces an RC4-encrypted exchange, which can stand out where the domain is AES-only or where RC4 downgrade is monitored.
- Supplying the **AES256** key produces an AES exchange that matches normal clients. Recover AES keys from LSASS (`sekurlsa::ekeys`) or from an `-just-dc` NTDS dump and prefer them.

## Exploitation notes

- Overpass-the-hash plus the AES key is the stealthy default for reusing a recovered machine or service secret as Kerberos.
- A TGT obtained this way drives everything downstream: [pass-the-ticket](pass-the-ticket.md), S4U [delegation](delegation/index.md) abuse, and service-ticket requests.
- Clock skew matters: Kerberos rejects tickets outside a ~5-minute window, so sync to the DC's time before requesting.

## Tools

- **Impacket** (`getTGT.py`, `-k`): request and use TGTs from hash or AES key.
- **Rubeus** (`asktgt`): on-host TGT request with `/rc4`, `/aes256`, `/ptt`.
- **Mimikatz** (`sekurlsa::ekeys`): recover AES keys from LSASS for pass-the-key.

## References

- [GhostPack Rubeus (asktgt)](https://github.com/GhostPack/Rubeus)
- [Impacket getTGT](https://github.com/fortra/impacket)
- [Microsoft: Kerberos authentication](https://learn.microsoft.com/en-us/windows-server/security/kerberos/kerberos-authentication-overview)
