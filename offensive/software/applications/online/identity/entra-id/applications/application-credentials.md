---
title: "Application credentials: adding a secret or certificate"
order: 4
description: "Adding secrets or certificates to an app or service principal you can write to, then authenticating as that identity with its Graph permissions."
keywords:
  - application credentials
  - addPassword
  - addKey
  - client secret
  - certificate
---

# Application credentials

If you can write to an application or service principal (as owner, or with the right directory role or Graph permission), you add your own **client secret or certificate** and then authenticate as that identity. You inherit everything it holds, and the credential persists independently of any user.

## Add a credential and log in

```bash
# add a client secret to an app registration
az ad app credential reset --id <appId> --append
# or via Graph: POST /applications/{id}/addPassword  (or addKey for a certificate)

# authenticate as the application
az login --service-principal -u <appId> -p <secret> --tenant <tenant>
```

## Exploitation notes

- This is the standard escalation after any app-write primitive (ownership, Application Administrator role, or `Application.ReadWrite` Graph permission).
- An added secret or certificate is durable persistence: it survives user password resets and is easy to miss among legitimate app credentials.
- Target apps that already hold privileged Graph app roles so the new credential inherits them ([API permissions](api-permissions.md)).

## Tools

- **az cli** (`az ad app credential reset`).
- **BARK** (`New-AppRegSecret` / `New-SPSecret`): automate credential addition.

## References

- [SpecterOps: BARK](https://github.com/BloodHoundAD/BARK)
- [HackTricks Cloud: application credentials](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: application and service principal objects](https://learn.microsoft.com/entra/identity-platform/app-objects-and-service-principals)
