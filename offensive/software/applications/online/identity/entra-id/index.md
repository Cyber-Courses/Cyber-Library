---
title: "Entra ID"
description: "Attacking Microsoft Entra ID (Azure AD) tenants: authentication and token abuse, application and service-principal takeover, directory-role and group privilege escalation, device registration, and cross-tenant and guest access."
keywords:
  - Entra ID
  - Azure AD
  - tenant
  - service principal
  - PRT
---

# Entra ID

Microsoft Entra ID (formerly Azure AD) is the cloud **directory** behind Microsoft 365 and Azure: the tenant-wide identity plane of users, groups, applications, service principals, and devices, reached through the Microsoft Graph and the sign-in endpoints. It is attacked like a directory, not like a cloud resource platform, which is why it sits here next to Active Directory rather than under Cloud.

Entra attacks move from outside-in: enumerate the tenant and its users, get a first token through spraying or phishing, then escalate through over-permissioned applications, directory roles, groups, and devices until you hold Global Administrator or an application that does. The currency throughout is the **token** (access, refresh, and the primary refresh token), not a password.

## Boundary with the Azure resource plane

This area is the **tenant and directory** plane. The Azure **resource** plane (subscriptions, VMs, storage, Azure RBAC, managed identities) is attacked separately and lives under [Cloud > Azure](../../../online/cloud/azure/index.md). The two meet at the `elevateAccess` seam, where a Global Administrator grants itself User Access Administrator over all subscriptions; that crossing is noted where it belongs. The on-premises bridge (Entra Connect, AD FS, PRT, pass-through auth) lives with [hybrid identity](../../../server/directory/active-directory/trusts/entra-hybrid.md).

## Enumeration

Enumeration folds into each surface, but the tenant-wide reconnaissance is run first: unauthenticated tenant and user discovery with **AADInternals** and the `GetCredentialType` and OpenID configuration endpoints, then authenticated enumeration of users, groups, apps, service principals, and roles with **ROADtools** (`roadrecon`) and **AzureHound** for the attack graph.

## Sections

- **[Authentication](authentication/index.md)**: user and tenant enumeration, password spraying, device-code and token phishing, primary refresh tokens, and conditional-access and MFA bypass.
- **[Applications](applications/index.md)**: consent phishing, Microsoft Graph permission abuse, added and federated credentials, and service-principal and ownership takeover.
- **[Roles](roles/index.md)**: privileged directory roles, PIM activation, and administrative-unit scoping.
- **[Groups](groups/index.md)**: dynamic-membership rule injection, group ownership, and role-assignable groups.
- **[Devices](devices/index.md)**: rogue device registration and join, bulk-enrollment tokens, and hybrid-join forgery for a PRT.
- **[Tenant](tenant/index.md)**: B2B guest access and cross-tenant access policies.

## References

- [HackTricks Cloud: Azure and Entra](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [AADInternals (Dr. Nestori Syynimaa)](https://aadinternals.com/aadinternals/)
- [ROADtools (Dirk-jan Mollema)](https://github.com/dirkjanm/ROADtools)
- [SpecterOps: AzureHound and BARK](https://github.com/BloodHoundAD/AzureHound)
