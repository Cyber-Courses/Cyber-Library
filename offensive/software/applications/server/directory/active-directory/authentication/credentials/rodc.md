---
title: "RODC: cached credentials and the scoped golden ticket"
description: "Compromising a Read-Only Domain Controller to extract the credentials its Password Replication Policy allows it to cache, and forging a scoped golden ticket with the RODC's own krbtgt key that a writable DC will accept for the allowed principals."
keywords:
  - RODC
  - Password Replication Policy
  - msDS-RevealedUsers
  - RODC krbtgt
  - golden ticket
---

# RODC

A **Read-Only Domain Controller** holds a filtered, read-only copy of the directory and caches only the credentials its **Password Replication Policy (PRP)** permits. RODCs are often deployed in branch offices and physically or administratively **less protected** than writable DCs, so they are a soft route to the credentials they do hold, and to a limited but real ticket-forging primitive.

## What an RODC caches

- `msDS-RevealOnDemandGroup` is the **allow** list (accounts whose passwords the RODC may cache); `msDS-NeverRevealGroup` is the **deny** list.
- By default only the **RODC computer account** and the **RODC krbtgt** (`krbtgt_<number>`) are cached; everything else is cached on demand when an allowed principal authenticates.
- `msDS-RevealedList` on the RODC object records which accounts have actually been cached, which is a ready-made target list.

## Extracting cached credentials

With admin on the RODC, dump the secrets it has cached locally for every allowed principal:

```bash
# an RODC does not replicate outward, so DRSUAPI (-just-dc) fails; read its LOCAL NTDS.dit
secretsdump.py -just-dc -use-vss 'EXAMPLE/rodc-admin@<rodc>'   # VSS snapshot of the local DB
# or copy ntds.dit + the SYSTEM hive off the box and extract offline:
secretsdump.py -ntds ntds.dit -system SYSTEM LOCAL
```

## The RODC golden ticket

The RODC has its **own krbtgt key**. Forge a TGT with it, and a **writable** DC will honour that ticket, but only for principals the RODC is allowed to reveal (in `msDS-RevealOnDemandGroup`, not in `msDS-NeverRevealGroup`) and only with the correct **key version number**:

```bash
# Rubeus forges the RODC branch (sets the RODC number and matching kvno); plain ticketer.py
# hardcodes kvno 2 and cannot set the RODC number, so a writable DC rejects its ticket
Rubeus.exe golden /rc4:<rodc-krbtgt-hash> /rodcNumber:<N> /user:allowed_user /id:<rid> \
  /domain:example.local /sid:<sid> /groups:<allowed>
```

## Exploitation notes

- The payoff scales with the **PRP**: a policy that allows caching a privileged account (or an over-broad `Allowed RODC Password Replication Group`) turns RODC compromise into compromise of that account, and sometimes Domain Admin.
- The RODC golden ticket is **scoped**, not a full golden ticket: it works only for the allowed set, so enumerate `msDS-RevealOnDemandGroup` before forging.
- RODCs sit at the edge of Tier Zero and are frequently managed by non-Tier-0 admins, so the `managedBy`/local-admin delegation on the RODC object is itself a path in.
- Cached hashes feed [pass-the-hash](../ntlm/pass-the-hash.md) and the recovered RODC krbtgt feeds the scoped [forged ticket](../kerberos/forged-tickets.md).

## Tools

- **Impacket `secretsdump.py`** (`-use-vss` or offline `-ntds ... LOCAL`): dump the RODC's locally cached secrets.
- **Rubeus** (`golden /rodcNumber`): forge the RODC-scoped TGT with the correct RODC number and kvno.
- **mimikatz**: local secret extraction on the box.

## References

- [adsecurity.org: attacking Read-Only Domain Controllers](https://adsecurity.org/?p=3592)
- [SpecterOps: at the edge of Tier Zero, the curious case of the RODC](https://posts.specterops.io/at-the-edge-of-tier-zero-the-curious-case-of-the-rodc-ef5f1799ca06)
- [The Hacker Recipes: RODC](https://www.thehacker.recipes/ad/movement/builtins/rodc)
