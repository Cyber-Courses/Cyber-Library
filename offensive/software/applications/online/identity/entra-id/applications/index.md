---
title: "Entra applications"
order: 2
description: "Abusing Entra app registrations and service principals: consent phishing, Microsoft Graph permission abuse, added credentials, federated identity credentials, and ownership takeover."
keywords:
  - app registration
  - service principal
  - consent phishing
  - Graph API
  - application credentials
---

# Applications

App registrations and their tenant instances, **service principals**, are first-class identities in Entra, and they are routinely over-permissioned. Control of an app that holds a high-value Microsoft Graph permission is equal to holding that permission yourself, which makes applications a primary escalation and persistence surface.

## What folds in here

- **[Service principals](service-principals.md)**: enumerating and hijacking app identities and their app roles.
- **[Consent phishing](consent-phishing.md)**: luring users or admins into granting a malicious OAuth app.
- **[API permissions](api-permissions.md)**: abusing Microsoft Graph application permissions to reach tenant control.
- **[Application credentials](application-credentials.md)**: adding a secret or certificate to authenticate as an app.
- **[Federated credentials](federated-credentials.md)**: adding a federated identity credential to mint tokens without a secret.
- **[Ownership](ownership.md)**: abusing app or service-principal ownership to take it over.

## References

- [SpecterOps: BARK (Azure app attack paths)](https://github.com/BloodHoundAD/BARK)
- [GraphRunner (dafthack)](https://github.com/dafthack/GraphRunner)
- [HackTricks Cloud: Azure applications](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
