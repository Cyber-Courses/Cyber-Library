---
title: "Privilege escalation"
order: 2
description: "Climbing Azure RBAC: writing role assignments to Owner, crafting permissive custom roles, elevate-access to root, assigning privileged managed identities, and deploying templates that run as a higher identity."
keywords:
  - Azure privilege escalation
  - roleAssignments write
  - custom role
  - elevateAccess
  - managed identity
  - ARM deployment
---

# Privilege escalation

Azure escalation is an authorization-plane problem. A principal that can write the RBAC graph, or that controls a resource carrying a managed identity, promotes itself without touching a host. The paths below are the Azure resource-plane equivalents of the AWS IAM catalog, grounded in the NetSPI MicroBurst and SpecterOps BARK research.

## The paths

- **[Role assignment write](role-assignment-write.md)**: `Microsoft.Authorization/roleAssignments/write` to grant yourself Owner or Contributor at any scope.
- **[Custom role definition](custom-role-definition.md)**: `roleDefinitions/write` to craft a role with wildcard actions, then self-assign it.
- **[Elevate access](elevate-access.md)**: toggling User Access Administrator at the root scope to reach every subscription in the tenant.
- **[Managed identity assignment](managed-identity-assignment.md)**: attaching a privileged managed identity to a resource you control, then minting its token.
- **[Deployment template](deployment-template.md)**: an ARM deployment or deploymentScripts resource that runs as a privileged identity.

## Choosing a path

A `roleAssignments/write` is the most direct lever when you hold it. Where you do not, control of a resource that already carries a privileged managed identity (a VM, a Function, an Automation Account) is usually the way up: borrow the identity through [managed identities](../managed-identities/index.md) and the compute surface. BARK enumerates which of these the current principal can reach.

## References

- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
- [SpecterOps: Azure privilege escalation with BARK](https://github.com/BloodHoundAD/BARK)
- [HackTricks Cloud: Azure privilege escalation](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-privilege-escalation/index.html)
