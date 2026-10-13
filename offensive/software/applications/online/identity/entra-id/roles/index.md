---
title: "Entra roles"
order: 3
description: "Escalating directory privilege in Entra: abusing privileged roles such as Privileged Role Administrator and Application Administrator, PIM activation, and administrative-unit scoping."
keywords:
  - Entra roles
  - privilege escalation
  - PIM
  - Global Administrator
  - administrative units
---

# Roles

Entra directory roles grant tenant-wide administrative power, and several intermediate roles reach Global Administrator through a short chain: a role that can assign roles, reset a privileged user's credentials, or add an application secret is effectively Global Admin. This surface is about recognizing those roles and the activation and scoping machinery around them.

## What folds in here

- **[Privilege escalation](privilege-escalation.md)**: the roles that chain to Global Administrator.
- **[PIM](pim.md)**: activating eligible privileged roles just in time.
- **[Administrative units](administrative-units.md)**: scoped role assignments and their gaps.

## References

- [SpecterOps: AzureHound role attack paths](https://github.com/BloodHoundAD/AzureHound)
- [HackTricks Cloud: Entra roles](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: Entra built-in roles](https://learn.microsoft.com/entra/identity/role-based-access-control/permissions-reference)
