---
title: "Azure compute"
order: 3
description: "Attacking Azure compute: VM run command and extensions, managed identities, AKS cluster access and node identity, Container Registry images and tasks, and Container Instances and scale sets."
keywords:
  - Azure compute
  - virtual machines
  - run command
  - AKS
  - managed identity
---

# Compute

Azure compute is attacked two ways: run code on a resource to reach the host, and steal the **managed identity** the resource carries to reach the subscription. A VM, container, or cluster you can write to runs your code through a first-class management operation (run command, an extension, `command invoke`), and the token minted for its attached identity is usually worth more than the host.

## Surfaces

- **[Virtual machines](virtual-machines/index.md)**: run command, Custom Script Extension, managed-identity theft, and user data.
- **[AKS](aks/index.md)**: admin credentials, `command invoke`, and node-pool identity.
- **[Container Registry](container-registry/index.md)**: admin credentials, ACR Tasks as a managed identity, and image poisoning.
- **[Container Instances](container-instances.md)**: running a container as an attached identity.
- **[Scale sets](scale-sets.md)**: run command and extensions across every instance.

## References

- [HackTricks Cloud: Azure compute](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
- [Microsoft: Azure VM run command](https://learn.microsoft.com/azure/virtual-machines/run-command-overview)
