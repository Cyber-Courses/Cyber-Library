---
title: "Managed identities"
description: "Abusing Azure managed identities: minting tokens for the system-assigned and user-assigned identities bound to compute, functions, and other resources."
keywords:
  - managed identity
  - system-assigned
  - user-assigned
  - IMDS
  - token
  - Azure AD
---

# Managed identities

A managed identity is an Entra service principal that Azure binds to a resource and whose credentials the platform rotates. Any code running on the resource can ask the local metadata endpoint for a token for that identity, so controlling the resource means acting as the identity with all of its RBAC. Managed identities are the engine behind most Azure escalation and lateral movement.

## The two kinds

- **[System-assigned](system-assigned.md)**: created with and tied to one resource, deleted with it.
- **[User-assigned](user-assigned.md)**: a standalone identity that can be attached to many resources, so its compromise is reusable.

Obtaining the token is a [credentials](../../credentials/instance-metadata/index.md) technique (the IMDS endpoint); attaching a privileged identity to a resource you control is the [managed identity assignment](../privilege-escalation/managed-identity-assignment.md) escalation.

## References

- [SpecterOps: managed identity attack paths (BARK)](https://github.com/BloodHoundAD/BARK)
- [Microsoft: managed identities overview](https://learn.microsoft.com/entra/identity/managed-identities-azure-resources/overview)
- [HackTricks Cloud: Azure managed identities](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
