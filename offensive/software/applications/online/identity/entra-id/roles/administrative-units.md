---
title: "Administrative units: scoped-role abuse"
description: "Abusing administrative units: scoped role assignments and restricted-management AU gaps that delegate control over subsets of the directory."
keywords:
  - administrative units
  - scoped roles
  - delegated administration
  - restricted management
  - AU
---

# Administrative units

An administrative unit scopes a directory role to a subset of users, groups, or devices. A scoped **User Administrator** or **Authentication Administrator** over an AU can reset passwords or MFA for every member of that AU, which is a strong escalation if a privileged user happens to fall inside it, and a quiet one because the role looks limited.

## Abuse a scoped role

```bash
# enumerate AUs and their scoped role assignments and members
az rest --method GET --url "https://graph.microsoft.com/v1.0/directory/administrativeUnits"
az rest --method GET --url "https://graph.microsoft.com/v1.0/directory/administrativeUnits/<au>/members"
# as a scoped User Administrator, reset a member's password / auth methods
```

## Exploitation notes

- A scoped Authentication Administrator over an AU that contains a privileged user can reset that user's credentials, escalating beyond the AU's apparent limit.
- Restricted-management AUs are meant to protect members, but misconfigured scoping and overlapping assignments create gaps worth enumerating.
- Scoped roles read as low-risk in a role review, so they are a good quiet foothold.

## Tools

- **az cli** / **Graph** (`directory/administrativeUnits`).
- **ROADtools** / **AzureHound**: AU and scoped-assignment graph.

## References

- [HackTricks Cloud: administrative units](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [SpecterOps: AzureHound](https://github.com/BloodHoundAD/AzureHound)
- [Microsoft: administrative units](https://learn.microsoft.com/entra/identity/role-based-access-control/administrative-units)
