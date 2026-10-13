---
title: "Enumeration: tenant, users, and federation without credentials"
order: 1
description: "Enumerating Entra users, tenant details, and federation configuration without credentials: GetCredentialType, OneDrive and autodiscover probing, and AADInternals tenant recon."
keywords:
  - user enumeration
  - tenant enumeration
  - GetCredentialType
  - AADInternals
  - federation
---

# Enumeration

Entra leaks a surprising amount before authentication. The login service confirms whether a username exists and whether its domain is managed or federated, and open tenant metadata reveals the tenant ID, domains, and branding. This builds the user list and tells you which sign-in path to attack.

## Tenant and domain recon

```powershell
# AADInternals: tenant details, domains, and federation status from outside
Invoke-AADIntReconAsOutsider -DomainName example.com
```

```bash
# tenant id and endpoints from the OpenID configuration
curl -s https://login.microsoftonline.com/example.com/.well-known/openid-configuration
# federation / managed status for a user
curl -s "https://login.microsoftonline.com/common/GetCredentialType" \
  -H 'Content-Type: application/json' \
  -d '{"Username":"user@example.com"}'
```

## User validation

`GetCredentialType` (and OneDrive or Autodiscover probing) returns an `IfExistsResult` that confirms valid usernames without a sign-in attempt, so a name list is validated quietly before any spray.

## Exploitation notes

- A `IfExistsResult` of 0 means the account exists; harvest a validated list to feed [password spraying](password-spraying.md).
- Federated domains send authentication to an external IdP (AD FS), which changes the spray target and lockout behaviour.
- None of this consumes a login attempt, so it does not trip Smart Lockout or alerts.

## Tools

- **AADInternals** (`Invoke-AADIntReconAsOutsider`): outside-in tenant recon.
- **o365spray** / **TeamFiltration**: username validation and OSINT.

## References

- [AADInternals: outsider recon](https://aadinternals.com/aadinternals/)
- [TeamFiltration (Flangvik)](https://github.com/Flangvik/TeamFiltration)
- [HackTricks Cloud: Azure enumeration](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
