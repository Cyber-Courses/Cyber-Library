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
# cached domain credentials from a compromised RODC (its local copy, incl. the RODC krbtgt)
secretsdump.py 'EXAMPLE/rodc-admin@<rodc>' -just-dc
# or lsadump / the registry hives on the box
```

## The RODC golden ticket

The RODC has its **own krbtgt key**. Forge a TGT with it, and a **writable** DC will honour that ticket, but only for principals the RODC is allowed to reveal (in `msDS-RevealOnDemandGroup`, not in `msDS-NeverRevealGroup`) and only with the correct **key version number**:

```bash
# forge with the RODC krbtgt key and its kvno; usable for allowed principals against a writable DC
ticketer.py -nthash <rodc-krbtgt-hash> -domain-sid <sid> -domain example.local \
  -user-id 1137 -groups <allowed> allowed_user
```

## Exploitation notes

- The payoff scales with the **PRP**: a policy that allows caching a privileged account (or an over-broad `Allowed RODC Password Replication Group`) turns RODC compromise into compromise of that account, and sometimes Domain Admin.
- The RODC golden ticket is **scoped**, not a full golden ticket: it works only for the allowed set, so enumerate `msDS-RevealOnDemandGroup` before forging.
- RODCs sit at the edge of Tier Zero and are frequently managed by non-Tier-0 admins, so the `managedBy`/local-admin delegation on the RODC object is itself a path in.
- Cached hashes feed [pass-the-hash](../ntlm/pass-the-hash.md) and the recovered RODC krbtgt feeds the scoped [forged ticket](../kerberos/forged-tickets.md).

## Tools

- **Impacket** (`secretsdump.py`, `ticketer.py`): dump cached secrets and forge the RODC-scoped TGT.
- **mimikatz / Rubeus**: local secret extraction and ticket forging on Windows.

## References

- [adsecurity.org: attacking Read-Only Domain Controllers](https://adsecurity.org/?p=3592)
- [SpecterOps: at the edge of Tier Zero, the curious case of the RODC](https://posts.specterops.io/at-the-edge-of-tier-zero-the-curious-case-of-the-rodc-ef5f1799ca06)
- [The Hacker Recipes: RODC](https://www.thehacker.recipes/ad/movement/builtins/rodc)
