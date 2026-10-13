---
title: "Ownership: taking over an app through its owner"
order: 6
description: "Abusing app and service-principal ownership: an owner can add credentials and change configuration to take over the application identity."
keywords:
  - application ownership
  - owner
  - service principal
  - credentials
  - takeover
---

# Ownership

An application or service-principal **owner** can modify the object, which includes adding credentials. So ownership of a privileged app is a full takeover: add a secret, authenticate as the app, and inherit its permissions. Ownership is handed out casually and rarely reviewed, so it is a common quiet edge to a powerful app.

## Abuse ownership

```bash
# list apps you own (or add yourself as owner where you hold the rights)
az ad app owner list --id <appId>
az ad app owner add --id <appId> --owner-object-id <you>

# as owner, add a credential and log in as the app
az ad app credential reset --id <appId> --append
```

## Exploitation notes

- Ownership plus [application credentials](application-credentials.md) is the chain: owner adds a secret, then authenticates as the app.
- Target owners of apps holding dangerous Graph app roles; the owner edge turns into those permissions.
- Adding yourself as owner (where a role or permission allows) is itself persistence, re-grantable after a credential is cleaned up.

## Tools

- **az cli** (`az ad app owner`).
- **BARK** / **AzureHound**: owner-edge attack paths.

## References

- [SpecterOps: AzureHound owner edges](https://github.com/BloodHoundAD/AzureHound)
- [HackTricks Cloud: application ownership](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: app registration owners](https://learn.microsoft.com/entra/identity-platform/app-objects-and-service-principals)
