---
title: "Bastion: pivoting into private VMs"
order: 6
description: "Abusing Azure Bastion as a pivot into private VMs over RDP and SSH."
keywords:
  - Bastion
  - pivot
  - RDP
  - SSH
  - private VM
---

# Bastion

Azure Bastion is a managed jump host that brokers RDP and SSH to VMs that have no public IP, straight from the portal or the CLI over the control plane. For an attacker holding a token with rights over Bastion and a target VM, it is a sanctioned pivot into the private network that needs no inbound NSG rule and leaves the VM with no public exposure to find.

## Connecting through Bastion

```bash
az network bastion list -o table
# tunnel to a private VM's RDP/SSH port through Bastion over the control plane
az network bastion tunnel --name <bastion> -g <rg> \
  --target-resource-id <vm-resource-id> --resource-port 22 --port 50022
# then ssh/rdp to localhost:50022
```

## Exploitation notes

- Reaching a VM this way needs `Microsoft.Network/bastionHosts/*` read plus `Reader` on the VM and the action to start a tunnel or session; a role that grants Bastion connect is a quiet path onto otherwise unreachable hosts.
- The connection is a legitimate management action, so it blends into normal operator traffic far better than opening an NSG port.
- Once on the VM, pull its [managed identity](../compute/virtual-machines/managed-identity.md) token and continue from the host.

## Tools

- **az network bastion tunnel / ssh / rdp**: broker the connection from the CLI.
- **Azure portal**: the same pivot through the browser session.

## References

- [HackTricks Cloud: Azure services](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: Azure Bastion](https://learn.microsoft.com/azure/bastion/bastion-overview)
- [MicroBurst (NetSPI)](https://github.com/NetSPI/MicroBurst)
