---
title: "Trust enumeration: mapping domain and forest trusts"
description: "Enumerating Active Directory trust relationships: intra-forest and cross-forest trusts, their direction and transitivity, and the attributes that decide whether a trust is abusable for lateral movement."
keywords:
  - trust enumeration
  - domain trust
  - forest trust
  - trustAttributes
  - SID filtering
---

# Trust enumeration

Trusts let principals in one domain authenticate to resources in another. For an attacker they are the map of where a compromise can spread beyond the current domain, up to the forest root and across to other forests. Enumeration here establishes which trusts exist, their direction and type, and whether their configuration permits the cross-boundary attacks.

## Listing trusts

Trusts are stored as `trustedDomain` objects and are readable by any domain user:

```
# PowerView maps trusts and can walk them recursively
Get-DomainTrust
Get-DomainTrustMapping            # follow trusts across the environment
Get-ForestDomain -Forest example.local
```

```bash
# From a domain host / Linux
nltest /domain_trusts /all_trusts
nxc ldap <dc> -u user -p pass -M enum_trusts
ldapsearch ... '(objectClass=trustedDomain)' name trustDirection trustType trustAttributes
```

## Reading the trust

Three attributes decide abusability:

- **`trustDirection`**: inbound, outbound, or bidirectional. Attacks flow along the direction of trust (you abuse a trust that trusts *your* domain).
- **`trustAttributes`**: this is the attribute that defines the topology, through flags such as `WITHIN_FOREST` (an intra-forest parent-child or tree-root trust), `FOREST_TRANSITIVE` (a forest trust), and whether **SID filtering** / quarantine (`TREAT_AS_EXTERNAL`, `QUARANTINED_DOMAIN`) is enabled. SID filtering is the control that blocks SID-history injection across the trust, so its presence or absence decides whether the cross-forest escalation is viable.
- **`trustType`**: the trust *protocol and partner type* (an up-level Windows/AD trust versus, for example, an MIT Kerberos realm), not the parent-child versus external versus forest distinction. Intra-forest, external, and forest trusts can all report the same up-level type, so read the topology from `trustAttributes` above, not from `trustType`.

## Intra-forest versus cross-forest

The distinction is central:

- **Intra-forest** trusts (parent-child, tree-root) share the forest's `krbtgt` chain and, crucially, do **not** filter the `Enterprise Admins` SID, so compromising any domain in a forest leads to the forest root. The forest, not the domain, is the real security boundary.
- **Cross-forest** trusts are the security boundary; SID filtering is enabled by default, so escalation across them requires a weaker configuration or a different primitive.

## Exploitation notes

- Map the full trust graph before planning movement; `Get-DomainTrustMapping` and BloodHound's trust edges reveal reachable domains you may not have known existed.
- Record whether each trust is transitive and whether SID filtering is on, since those two facts gate the SID-history and ticket-based cross-domain attacks covered in the Trusts section.
- Foreign security principals (members of a domain's groups that live in another domain) are a quiet indicator of a usable trust relationship worth examining.

## Tools

- **PowerView `Get-DomainTrust` / `Get-DomainTrustMapping`**: trust discovery and walking.
- **nltest /domain_trusts**: native trust listing.
- **BloodHound**: trust edges across domains and forests.

## References

- The Hacker Recipes: Trusts
- Microsoft: Active Directory trust types and attributes
