---
title: "Assets and attack surface: What you protect, and where it can be reached"
description: "Assets are the things worth protecting, and the attack surface is the sum of all points where those assets can be reached or attacked."
keywords:
  - assets
  - attack surface
  - attack surface reduction
  - asset inventory
  - exposure
  - security fundamentals
---

# Assets and attack surface

**An asset is anything of value that an organization needs to protect, and the attack surface is the total set of points where those assets can be reached, probed, or attacked.** The two ideas are a matched pair: you cannot defend what you have not identified as worth defending, and you cannot measure your exposure until you know where an attacker can reach in. Risk only exists where a valuable asset meets a way to get at it, which is why nearly every security decision starts with these two questions.

## Deciding what counts as an asset

An asset is not only data. Security work typically groups assets into a few kinds, and each kind fails in its own way when it is lost or misused:

| Kind of asset | Examples | What losing it looks like |
| --- | --- | --- |
| Information | Customer records, source code, credentials, financial data, intellectual property | Leaked, altered, or held for ransom |
| Systems and software | Servers, laptops, mobile devices, applications, and the services that run them | Taken over, disrupted, or used as a foothold |
| Infrastructure and accounts | Networks, cloud tenants, domains, and the identities that control them | Hijacked control over everything built on top |
| People and reputation | Staff knowledge, and the trust an organization depends on to operate | Manipulated insiders, lost customers and partners |

The useful test is value: if its confidentiality, integrity, or availability matters to the organization, it is an asset (see the [CIA triad](cia-triad.md)). Credentials and identities deserve special attention, because compromising one often opens a path to many other assets at once.

## Mapping where each asset can be reached

The attack surface is every place where one of those assets is exposed to interaction. NIST defines it as the set of points on the boundary of a system or environment where an attacker can try to enter, cause an effect, or extract data. In practice those points fall into a few families:

- **Network:** each open port, exposed service, and remote access gateway.
- **Software:** login pages, API endpoints, file upload forms, and any input an application accepts.
- **Physical:** USB slots, unattended workstations, and devices that can be carried away.
- **Third parties:** integrations, suppliers, and vendor accounts that hold a path into your environment.
- **Human:** each person who can be phoned, emailed, or phished, sometimes called the human attack surface. Even a published staff email address is a point on the surface.

## One database, three exposures

Consider a single customer database, and how its exposure changes with what surrounds it:

1. **Isolated on an internal network with no inbound path.** The surface is small: an attacker must first get inside the network, and only a few administrators can reach the database.
2. **Behind a public web form.** The same data is now reachable by anyone on the internet who can submit input, so every flaw in the form and the application behind it becomes part of the surface.
3. **Also synced to a vendor's analytics platform.** The asset now has a second home, and the vendor's staff, accounts, and security practices join the surface, even though nothing changed on your own servers.

The data is the same in all three cases; only the reachability changed. A larger surface means more points a defender must patch, monitor, and reason about, and more points an attacker can study for [vulnerabilities](vulnerabilities-and-exploits.md).

## Shrinking the surface without breaking the business

Reducing the surface lowers risk directly: closing an unused port or removing a forgotten server takes an entire class of attacks off the table without any need to detect them. Teams usually work through it in this order:

1. **Inventory what exists.** Map assets and what they matter for, because unknown assets are defended by accident, if at all.
2. **Remove what is not needed.** Close unused services, retire old systems, and delete stale accounts and integrations.
3. **Restrict reach to what remains.** Limit who and what can connect to each asset, applying [least privilege](../principles/least-privilege.md) to access and keeping data collection to a [minimum](../principles/minimization.md).
4. **Guard the points that must stay open.** Place [controls and countermeasures](controls-and-countermeasures.md) on the exposed points that the business genuinely needs.

## The assets nobody remembered

The hardest part is usually not defending known assets but finding the ones nobody remembered they had: a test server left online, a cloud storage bucket created for a one-off project, a SaaS tool adopted by one team without IT's knowledge (shadow IT). Because cloud resources and new integrations appear faster than any one-time audit can keep up with, attack surface management is now a continuous discipline of discovery rather than a yearly checklist. [Threat actors](threats-and-threat-actors.md) scan for exactly these forgotten points, so an organization that does not look at its own surface from the outside leaves that view to its attackers.

## References

- [NIST Glossary: Attack Surface](https://csrc.nist.gov/glossary/term/attack_surface)
- [NIST Glossary: Asset](https://csrc.nist.gov/glossary/term/asset)
- [NIST SP 800-30 Rev. 1, Guide for Conducting Risk Assessments](https://csrc.nist.gov/pubs/sp/800/30/r1/final)
- [OWASP: Attack Surface Analysis Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Attack_Surface_Analysis_Cheat_Sheet.html)
- [CIS Control 1: Inventory and Control of Enterprise Assets](https://www.cisecurity.org/controls/inventory-and-control-of-enterprise-assets)
- [CISA: Foundations for OT Cybersecurity, Asset Inventory Guidance for Owners and Operators](https://www.cisa.gov/resources-tools/resources/foundations-ot-cybersecurity-asset-inventory-guidance-owners-and-operators)
