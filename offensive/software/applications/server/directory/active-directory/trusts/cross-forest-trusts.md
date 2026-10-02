---
title: "Cross-forest trusts: moving between forests"
description: "Moving across an Active Directory forest trust, where SID filtering and selective authentication are enforced by default: using access explicitly granted to foreign principals, trusts with SID filtering relaxed, and TGT delegation to unconstrained-delegation hosts across the trust."
keywords:
  - forest trust
  - SID filtering
  - selective authentication
  - TGT delegation
  - foreign security principal
---

# Cross-forest trusts

A forest trust connects two separate forests, and unlike an intra-forest trust it is meant to be a real boundary. By default it enforces **SID filtering** (quarantine), which strips SIDs that do not belong to the trusted forest, so the SID-history trick that crosses domains inside a forest does **not** cross a forest trust. Movement across a forest trust is therefore narrower and depends on what the trust and its administrators actually allow.

## What the guardrails do

- **SID filtering / quarantine**: removes foreign SIDs (including `extraSids`) from inbound tickets, so a privileged SID forged in forest A is stripped before forest B honours it.
- **Selective authentication**: when enabled, principals from the trusted forest get no access unless explicitly granted "Allowed to authenticate" on specific resources, tightening the boundary further.
- **Trust direction and transitivity**: a forest trust can be one-way and is non-transitive to third forests, limiting reach.

## Paths that still work

- **Access granted to foreign principals**: organisations add users or groups from the other forest to local groups and ACLs (foreign security principals). Those grants are legitimate and unfiltered, so enumerate what the trusted forest's principals can already reach and use it.
- **Relaxed SID filtering**: SID filtering can be disabled or loosened on a trust (for example to support migrations), re-opening SID-history injection across the trust. Read the trust's filtering posture during [trust enumeration](trust-enumeration.md); where it is off, treat the forest trust like an intra-forest one.
- **TGT delegation to unconstrained delegation**: if TGT delegation is enabled on the trust and the trusted forest has a host with [unconstrained delegation](../authentication/kerberos/delegation/unconstrained.md), coercing a privileged account across the trust to authenticate to that host captures its TGT, bridging the forests.

## Exploitation notes

- Start from what is **already granted**: cross-forest compromise usually comes from a misconfigured ACL or group membership spanning the trust, not from breaking SID filtering.
- SID filtering being **on** is the default and the thing to check first; a trust flagged with SID history enabled or quarantine off collapses the boundary to intra-forest difficulty.
- Selective authentication, where enabled, means even valid foreign credentials get nothing without a per-resource grant, so hunt for those grants specifically.

## Tools

- **BloodHound**: maps cross-forest ACLs, foreign-group memberships, and reachable principals.
- **Impacket / Rubeus**: request and use cross-forest tickets once a path exists.
- **nltest / Get-ADTrust**: read trust direction, transitivity, and filtering attributes.

## References

- The Hacker Recipes: cross-forest trusts
- Microsoft: SID filtering, quarantine, and selective authentication
