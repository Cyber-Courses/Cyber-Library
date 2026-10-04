---
title: "Azure identity"
description: "Abusing the Azure RBAC authorization plane: enumerating role assignments, escalating through role and custom-role writes, elevate-access, and managed-identity assignment, and activating privileged roles through PIM."
keywords:
  - Azure RBAC
  - role assignment
  - privilege escalation
  - managed identity
  - PIM
---

# Identity

Azure authorization is **Azure RBAC**: role assignments bind a principal to a role at a scope (management group, subscription, resource group, or resource). Attacks here are about turning a write over the authorization plane, or control of a resource that carries a managed identity, into a higher role. This is the Azure **resource plane** only; tenant and directory attacks (Entra roles, users, applications) live under [Entra ID](../../../directory/entra-id/index.md) in the Directory area.

The core moves are writing a role assignment to grant yourself Owner, crafting a custom role with a wildcard action, flipping on `elevateAccess` to seize User Access Administrator at the root, and attaching or borrowing a privileged **managed identity** to mint its token.

## What folds in here

- **[Enumeration](enumeration.md)**: mapping role assignments, custom roles, and managed identities with Resource Graph, az cli, and ROADtools.
- **[Privilege escalation](privilege-escalation/index.md)**: role-assignment and custom-role writes, elevate-access, managed-identity assignment, and privileged deployments.
- **[Managed identities](managed-identities/index.md)**: system- and user-assigned identities as the token source behind most Azure escalation.
- **[PIM](pim.md)**: activating eligible privileged roles through Privileged Identity Management.

Managed-identity **token retrieval** from the metadata endpoint is a credential technique and lives under [credentials](../credentials/instance-metadata/index.md); this surface is about obtaining and escalating the role itself.

## References

- [HackTricks Cloud: Azure privilege escalation](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-privilege-escalation/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
- [SpecterOps: Managed Identity attack paths (BARK)](https://github.com/BloodHoundAD/BARK)
