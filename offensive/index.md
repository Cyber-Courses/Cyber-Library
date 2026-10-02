---
title: "Offensive security"
description: Penetration testing, red teaming, bug bounty, and adversary emulation—methodology, tradecraft, and how this library maps attack-relevant topics.
keywords:
  - offensive security
  - penetration testing
  - red teaming
  - adversary emulation
  - bug bounty
  - social engineering
---

# Offensive

**Offensive** work means acting as an **authorized** adversary: you probe systems the way real attackers do—recon, exploitation, lateral movement, persistence—so weaknesses show up as **evidence** (reproducible steps, impact, blast radius), not as abstract “risk scores.”

This branch is written for **operators and readers of findings**: what to try, how techniques chain, and where the library keeps the detail (web code, injection, access control, workflow flaws, and more).

## Methodologies (how engagements differ)

| Style | What you optimize for |
|-------|------------------------|
| **Penetration test** | Time-boxed coverage, clear **PoCs**, severity-rated issues, often scoped systems. |
| **Red team** | Longer campaign, stealth and **detection** pressure, goals (e.g. domain objective) over raw CVE count. |
| **Bug bounty** | Public or private programs, out-of-scope rules, payout-driven **signal** on high-impact bugs. |
| **Assumed breach / tabletop** | Sometimes paired with offensive work; this library focuses on **technical** tradecraft more than IR exercises. |

Techniques evolve constantly; the library favors **durable patterns** (classes of bug and how code mishandles trust) over one-off exploit names.

## Where to start in this library

- **[Software](software/index.md)** — Vulnerabilities that live in **application code**, APIs, and how programs process untrusted input. Server-side **web** material is organized under *Software → Applications → Server → Web → Code*.

Physical, pure network, or org-policy topics may live elsewhere in the wider wiki; this subtree is biased toward **software and web application** offensive content.

## Scope

Use the linked pages for **authorized** testing, research on systems you own, CTFs, and training labs. Legal and policy boundaries are on you and your engagement rules—not this documentation.
