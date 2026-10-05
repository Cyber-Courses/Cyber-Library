---
title: "Container Registry"
description: "Abusing Azure Container Registry: admin credentials and repository access, ACR Tasks that run as a managed identity, and image poisoning."
keywords:
  - ACR
  - Container Registry
  - admin credentials
  - ACR Tasks
  - image poisoning
---

# Container Registry

Azure Container Registry holds the images that AKS, App Service, and Container Instances pull and run, which makes it both a credential store and a supply-chain foothold. The admin account and repository rights give read and write to images; **ACR Tasks** run build steps in-registry as a managed identity.

## Pages

- **[Registry access](registry-access.md)**: admin credentials and repository pull and push.
- **[Tasks](tasks.md)**: ACR Tasks running as a managed identity, and image poisoning.

## References

- [HackTricks Cloud: ACR](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: ACR authentication](https://learn.microsoft.com/azure/container-registry/container-registry-authentication)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
