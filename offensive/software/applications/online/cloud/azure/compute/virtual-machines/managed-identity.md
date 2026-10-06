---
title: "Managed identity: stealing the VM identity token from IMDS"
order: 3
description: "Stealing the token of a VM's attached managed identity from IMDS to act across the subscription."
keywords:
  - VM managed identity
  - IMDS
  - token
  - subscription
  - privilege escalation
---

# Managed identity

A VM with a system- or user-assigned **managed identity** exposes that identity's tokens on the instance metadata endpoint. Once you run code on the VM (through [run command](run-command.md), an [extension](custom-script-extension.md), or a shell), you mint a token for any Azure audience the identity can reach and act as it, which is frequently Contributor or Owner somewhere.

## Minting the token on the box

```bash
# management plane
curl -s -H Metadata:true \
 "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/"
# other audiences: https://graph.microsoft.com/ , https://vault.azure.net
# user-assigned: add &client_id=<clientId> or &mi_res_id=<resourceId>
```

Feed the returned bearer token to `az rest` or the ARM API and enumerate what the identity can do.

## Exploitation notes

- This is the Azure end of the metadata-token pattern; the retrieval mechanics (user-assigned selectors, App Service `IDENTITY_ENDPOINT`, SSRF) live under [credentials](../../credentials/instance-metadata/index.md).
- Attaching a privileged identity to a VM you control is itself an escalation, covered under [identity](../../identity/managed-identities/index.md).
- A user-assigned identity shared across many resources widens blast radius: one VM token acts wherever that identity is assigned.

## Tools

- **Azure CLI** (`az rest`, `az account get-access-token`) once the token is exported.
- **MicroBurst** / **ROADtools**: identity enumeration and token handling.

## References

- [Microsoft: managed identities and IMDS](https://learn.microsoft.com/azure/active-directory/managed-identities-azure-resources/how-to-use-vm-token)
- [SpecterOps: Managed Identity attack paths (BARK)](https://github.com/BloodHoundAD/BARK)
- [HackTricks Cloud: Azure managed identities](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
