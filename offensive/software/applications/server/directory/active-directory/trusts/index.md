---
title: "Trusts: abusing Active Directory domain and forest trusts"
description: "Abusing Active Directory trust relationships to move between domains and forests: using trust keys, SID history and cross-domain tickets inside a forest, and the narrower paths across forest trusts bounded by SID filtering."
keywords:
  - active directory trusts
  - forest trust
  - SID history
  - SID filtering
  - inter-realm TGT
---

# Trusts

A trust lets principals in one domain authenticate to resources in another. Trusts are what make a **forest** a single security boundary and what connect separate forests. For an attacker, a trust is a bridge: compromise of one domain frequently extends to the domains that trust it, and the rules that are supposed to contain that spread (SID filtering, selective authentication) are often weaker than assumed, especially **within** a forest.

## The trust landscape

- **Intra-forest trusts** (between domains in one forest) are, by design, a weak boundary: the forest is the real security boundary, so cross-domain escalation within a forest is usually achievable.
- **Cross-forest trusts** are meant to be stronger, bounded by **SID filtering**, but misconfiguration, SID-history allowances, and shared accounts open paths across them.
- **Trust direction and transitivity** decide who can reach whom; a trusted domain's compromise often flows into the trusting one.

## How trusts are abused

- **Trust keys**: the shared key of an inter-domain trust forges inter-realm referral tickets to move across the trust.
- **SID history**: injecting a privileged SID from the target domain into a ticket grants that domain's access where SID filtering does not strip it; within a forest it typically is not stripped.
- **Cross-domain tickets**: a golden/forged ticket in a child domain, carrying an Enterprise Admin SID, escalates to the forest root where intra-forest filtering is absent.

## Pages

- **[Trust enumeration](trust-enumeration.md)**: mapping trusts, their direction, transitivity, and filtering posture.

## References

- The Hacker Recipes: domain and forest trusts
- Microsoft: how Active Directory trusts work and SID filtering
