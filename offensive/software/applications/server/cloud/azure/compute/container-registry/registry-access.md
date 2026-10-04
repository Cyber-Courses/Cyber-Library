---
title: "Registry access: admin credentials and repository pull and push"
description: "Reaching an Azure Container Registry with admin account credentials or repository pull and push rights to read and tamper with images."
keywords:
  - ACR
  - admin account
  - repository
  - pull
  - push
  - registry
---

# Registry access

When the ACR **admin account** is enabled, a single username and password grant full pull and push to every repository, and the credential is readable from the management plane by anyone with the right. Even without it, `AcrPull`/`AcrPush` on the registry lets you read images (for baked-in secrets) and push tampered ones.

## Grab the admin credential and log in

```bash
az acr credential show -n <registry>          # username + two passwords
az acr login -n <registry>                    # or docker login <registry>.azurecr.io
az acr repository list -n <registry>
docker pull <registry>.azurecr.io/<repo>:<tag>
```

## Exploitation notes

- Pull every image and grep layers for secrets, tokens, and connection strings; build artifacts leak these constantly.
- `az acr credential show` works for anyone with `listCredentials` on the registry, so the admin account is a soft target even without Docker on the host.
- Pushing a poisoned tag over an image that AKS or App Service pulls is covered under [Tasks](tasks.md) and image poisoning.

## Tools

- **Azure CLI** (`az acr credential show`, `az acr repository`).
- **docker** / **crane** / **skopeo**: pull, inspect, and push images.

## References

- [Microsoft: ACR authentication](https://learn.microsoft.com/azure/container-registry/container-registry-authentication)
- [HackTricks Cloud: ACR](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
