---
title: "Identity: attacking vendor-hosted identity providers"
description: "Attacking cloud-hosted identity providers and directories: tenants reached through authentication and token abuse, OAuth application and consent abuse, directory-role and group privilege escalation, and cross-tenant access. There is no directory server to reach, only the provider's sign-in surface and APIs."
keywords:
  - cloud identity
  - identity provider
  - Entra ID
  - Azure AD
  - tenant
  - OAuth
---

# Identity

A vendor-hosted identity provider is a directory you cannot reach as a server: there is no domain controller to touch, only the provider's sign-in endpoints, token service, and management APIs. The attack model is therefore identity-first and token-centric: enumerate the tenant, obtain a principal through a sprayed credential or a phished token, then escalate through application consent, directory roles, and group membership, and cross tenant boundaries through guest and federation trust. It sits under Online because the directory runs on the vendor's infrastructure, but the techniques mirror the directory attacks under Server rather than the resource-plane work under Cloud.

## Triage

```bash
# Is a target tenant Microsoft Entra ID, and does it federate?
curl -s "https://login.microsoftonline.com/<domain>/.well-known/openid-configuration" | python3 -m json.tool | head
curl -s "https://login.microsoftonline.com/getuserrealm.srf?login=user@<domain>&xml=1"   # Managed vs Federated
```

A Microsoft tenant routes to [Entra ID](entra-id/index.md). The on-premises directory it may sync from (Active Directory) is a self-hosted target under [Directory](../../server/directory/index.md), and the sync server bridging the two is covered with the AD hybrid-identity pages.

## Subtopics

- **[Entra ID](entra-id/index.md)**: Microsoft Entra ID (Azure AD) tenants, from authentication and token abuse through application, role, group, device, and cross-tenant attacks.

## References

- [Microsoft Entra documentation](https://learn.microsoft.com/en-us/entra/)
- [MITRE ATT&CK: Cloud matrix](https://attack.mitre.org/matrices/enterprise/cloud/)
