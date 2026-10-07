---
title: "Trusts: moving across Active Directory domains and forests"
order: 4
description: "Active Directory trust relationships as a path between domains and forests: trust keys and inter-realm tickets, SID history inside a forest, and the narrower paths across forest trusts bounded by SID filtering."
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

## How trusts are crossed

- **SID history**: a privileged SID from the target domain, placed in a forged ticket's `extraSids`, grants that domain's access wherever SID filtering does not strip it; within a forest it typically is not stripped.
- **Trust keys**: the shared key of an inter-domain trust forges the inter-realm referral tickets that move a principal across the trust.
- **Guardrails**: cross-forest trusts enforce SID filtering and selective authentication by default, so crossing them relies on granted access or relaxed configuration rather than SID injection.

## Pages

- **[Trust enumeration](trust-enumeration.md)**: mapping trusts, their direction, transitivity, and filtering posture.
- **[Intra-forest trusts](intra-forest-trusts.md)**: reaching the forest root from any domain via SID history.
- **[Cross-forest trusts](cross-forest-trusts.md)**: moving between forests under SID filtering and selective authentication.
- **[Trust keys and inter-realm tickets](trust-keys.md)**: forging referral tickets from the trust key.
- **[Entra hybrid](entra-hybrid.md)**: pivoting on-premises AD to the Entra ID tenant and back through Entra Connect.

## References

- The Hacker Recipes: domain and forest trusts
- Microsoft: how Active Directory trusts work and SID filtering
