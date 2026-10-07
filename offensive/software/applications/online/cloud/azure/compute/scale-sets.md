---
title: "Scale sets: run command and extensions across every instance"
order: 5
description: "Running commands and extensions across an Azure VM Scale Set to reach every instance and its managed identity."
keywords:
  - VMSS
  - scale sets
  - run command
  - extension
  - managed identity
---

# Scale sets

A VM Scale Set (VMSS) is a fleet of identical VMs managed as one resource. The same run-command and extension rights that take over a single VM apply to the set, so one action reaches every instance, and the set's **managed identity** is shared across all of them.

## Execute across the fleet

```bash
az vmss run-command invoke -g <rg> -n <vmss> --instance-id '*' \
  --command-id RunShellScript --scripts "id"
# or install an extension on the model so new instances inherit it
az vmss extension set -g <rg> --vmss-name <vmss> \
  --name CustomScript --publisher Microsoft.Azure.Extensions \
  --settings '{"commandToExecute":"..."}'
```

## Exploitation notes

- Setting an extension on the VMSS **model** means every instance, including ones scaled out later, runs it: durable fleet-wide execution.
- The scale set's managed identity token is reachable from any instance through IMDS, same as a standalone [VM](virtual-machines/managed-identity.md).

## Tools

- **Azure CLI** (`az vmss run-command invoke`, `az vmss extension set`).
- **MicroBurst**: bulk command execution.

## References

- [Microsoft: VMSS extensions](https://learn.microsoft.com/azure/virtual-machine-scale-sets/overview)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
- [HackTricks Cloud: Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
