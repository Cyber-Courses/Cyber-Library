---
title: "Assets and attack surface: What security protects and where it can be reached"
description: "What security protects, and the sum of all points where those assets can be reached or attacked."
keywords:
  - assets
  - attack surface
  - attack surface reduction
  - asset inventory
  - exposure
  - reachability
---

# Assets and attack surface

**An asset is anything an organization would be harmed to lose, and the attack surface is the complete set of points where those assets can be reached or attacked.** The two ideas only make sense together. An asset with no way in is out of an attacker's reach, though an accident, a failure, or a fire can still harm it, and a way in that leads to nothing of value is not worth an attacker's time. Security lives in the overlap: a valuable thing, plus a path to it.

## The defender's list versus the attacker's map

A defender tends to think in terms of a list: the data, systems, and accounts the business depends on. An attacker thinks in terms of a map: every place that list can be touched from the outside. The gap between the two is where incidents happen. Anything on the defender's list but off the attacker's map is safely out of reach; anything on the attacker's map but off the defender's list is an exposure nobody is watching. The goal of this pairing is to make the two views line up, so that every valuable asset is accounted for and every reachable point is known.

## Four kinds of asset, four ways to lose them

An asset is rarely just a file. Security work groups them by kind, because each kind fails differently when its [confidentiality, integrity, or availability](cia-triad.md) is broken.

| Kind | Examples | Losing it looks like |
| --- | --- | --- |
| Information | Records, source code, secrets | Leaked, altered, or ransomed |
| Systems | Servers, laptops, applications | Hijacked or disrupted |
| Identities | Accounts, keys, domains | Control handed to an attacker |
| Trust | Staff knowledge, reputation | Insiders misled, customers lost |

Identities deserve the most care, because one stolen credential rarely stops at one asset.

## The boundary where attacks begin

NIST defines the attack surface as the set of points on the boundary of a system where an attacker can try to enter, cause an effect, or extract data. In practice those boundary points fall into a few families:

- **Network:** open ports, exposed services, remote-access gateways.
- **Software:** login pages, API endpoints, upload forms, any accepted input.
- **Physical:** USB ports, unattended machines, portable devices.
- **Human:** anyone who can be phished or phoned.
- **Supply chain:** vendors and integrations that hold a path inward.

Each point is somewhere a [vulnerability](vulnerabilities-and-exploits.md) could live and be found.

## Adding one feature, widening the surface

Watch how a single asset's exposure changes as the business grows around it. The customer database never moves, but its reachability does.

1. **Internal only.** The database sits on a private network. The surface is a handful of administrators.
2. **A public web form is added.** Anyone on the internet can now submit input, so every flaw in that form joins the surface.
3. **Analytics are outsourced.** A copy is synced to a vendor. The vendor's staff, accounts, and security now sit between an attacker and your data, with nothing changed on your own servers.

The data is identical in all three steps. Only the number of paths to it grew.

## Surfaces grow faster than inventories

The hardest asset to defend is the one nobody remembered: a test server left online, a storage bucket from an old project, a tool a team adopted without telling anyone. Cloud resources appear faster than any annual audit, which is why attack surface management is now continuous discovery rather than a checklist. Shrinking the surface, by closing unused services and applying [least privilege](../principles/least-privilege.md) to what remains, takes whole classes of attack off the table before any detective control is needed. [Threat actors](threats-and-threat-actors.md) scan for forgotten points constantly, so an organization that never looks at its own surface from the outside leaves that view entirely to them.

## References

- [NIST Glossary: Attack Surface](https://csrc.nist.gov/glossary/term/attack_surface)
- [NIST Glossary: Asset](https://csrc.nist.gov/glossary/term/asset)
- [NIST SP 800-30 Rev. 1, Guide for Conducting Risk Assessments](https://csrc.nist.gov/pubs/sp/800/30/r1/final)
- [OWASP: Attack Surface Analysis Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Attack_Surface_Analysis_Cheat_Sheet.html)
- [CIS Control 1: Inventory and Control of Enterprise Assets](https://www.cisecurity.org/controls/inventory-and-control-of-enterprise-assets)
