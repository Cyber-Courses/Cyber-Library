---
title: "Forged tickets: golden, silver, diamond, and sapphire"
description: "Forging Kerberos tickets from stolen keys: golden tickets from the krbtgt key, silver tickets from a service key, and the diamond and sapphire variants that modify real tickets to evade PAC-based detection."
keywords:
  - golden ticket
  - silver ticket
  - diamond ticket
  - sapphire ticket
  - krbtgt
---

# Forged tickets

Kerberos trusts a ticket because it can be decrypted with the right key and its PAC is signed by the KDC. If you hold the relevant key, you can **forge** a ticket with any identity and privileges you like, because you can produce exactly what the verifier expects. Which key you hold decides what you can forge and how broad it is.

## Golden ticket (the krbtgt key)

The `krbtgt` account's key encrypts and signs every TGT in the domain. With it (from [DCSync](../credentials/ntds-and-dcsync.md) or an NTDS dump), you forge a TGT for **any** user with **any** group membership, valid until the `krbtgt` password changes twice:

```bash
ticketer.py -nthash <krbtgt-hash> -domain-sid <sid> -domain example.local \
  -user-id 500 -groups 512,519,518,520 Administrator
export KRB5CCNAME=Administrator.ccache
```

A golden ticket is domain-wide, long-lived persistence and impersonation in one artifact. Forge the SID of a privileged group (512 Domain Admins, 519 Enterprise Admins) to act as that group.

## Silver ticket (a service key)

A silver ticket is a forged **service ticket (TGS)**, encrypted with a single **service account's** key (a cracked service password or a machine account's key). It never contacts the KDC, so it is quiet, but it only works against that one service:

```bash
ticketer.py -nthash <service-hash> -domain-sid <sid> -domain example.local \
  -spn cifs/server.example.local Administrator
```

Scope is the trade-off: a silver ticket for `cifs/host` grants file access to that host, a `host/` ticket grants scheduled-task/service execution, but nothing beyond the chosen SPN.

## Diamond and sapphire (evading PAC checks)

Golden and silver tickets build a PAC from scratch, which modern detections flag (impossible group combinations, missing fields, mismatched timestamps). The diamond and sapphire variants instead start from a **real** ticket:

- **Diamond ticket**: request a legitimate TGT, decrypt it with the `krbtgt` key, modify the PAC (add groups), and re-encrypt. The ticket's envelope is genuine, so it matches a real request.
- **Sapphire ticket**: like diamond, but populate the PAC with a real privileged user's PAC obtained via S4U/U2U, so every field is authentic rather than hand-built.

```bash
Rubeus.exe diamond /krbkey:<krbtgt-aes> /user:user /password:pass /ticketuser:Administrator /ticketuserid:500 /groups:512
```

## Exploitation notes

- Prefer **AES** keys when forging: an RC4 golden ticket in an AES domain stands out, while an AES forgery blends in.
- Silver tickets are the stealthiest option for a single target because they never touch a DC, ideal when you have one service/machine key and a specific goal.
- Golden-ticket persistence survives a single `krbtgt` reset; defenders must rotate `krbtgt` **twice** to invalidate it, which is why it is a favourite persistence primitive (see the Persistence section).
- A machine account's key forges silver tickets for that computer's services and underpins resource-based [delegation](delegation/resource-based-constrained.md) abuse.

## Tools

- **Impacket** (`ticketer.py`): golden and silver ticket forgery from hash or AES key.
- **Rubeus** (`golden`, `silver`, `diamond`): on-host forgery including diamond.
- **Mimikatz** (`kerberos::golden`): classic golden/silver forging.

## References

- [Impacket ticketer (golden / silver)](https://github.com/fortra/impacket)
- [GhostPack Rubeus (golden / silver / diamond)](https://github.com/GhostPack/Rubeus)
- [Microsoft (MS-PAC): the Kerberos PAC](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-pac/166d8064-c863-41e1-9c23-edaaa5f36962)
