---
title: "User data: reading cloud-init secrets or writing a boot script"
description: "Reading Azure VM user data and cloud-init for injected secrets, or writing it to run code at boot."
keywords:
  - user data
  - cloud-init
  - VM
  - boot script
  - secrets
---

# User data

Azure VMs carry **user data** (and cloud-init custom data) that provisioning scripts read at boot. Operators routinely leave secrets there, and the field is readable from the metadata endpoint on the box and from the management plane. Write access lets you plant a script that runs on the next boot.

## Reading it

```bash
# from the management plane
az vm show -g <rg> -n <vm> --query userData -o tsv | base64 -d
# from inside the VM via IMDS
curl -s -H Metadata:true "http://169.254.169.254/metadata/instance/compute/userData?api-version=2021-01-01&format=text" | base64 -d
```

## Writing a boot payload

```bash
az vm update -g <rg> -n <vm> --set userData="$(base64 -w0 payload.sh)"
# combine with a stop/start, or pair with cloud-init custom-data on create
```

## Exploitation notes

- Grep recovered user data and cloud-init for passwords, SAS tokens, and connection strings; it is a common store for them.
- Custom data is only processed at provisioning, so writing user data usually pairs with a reboot or the [Custom Script Extension](custom-script-extension.md) for immediate execution.

## Tools

- **Azure CLI** (`az vm show`, `az vm update`).
- **MicroBurst**: bulk VM data and secret collection.

## References

- [Microsoft: user data for Azure VMs](https://learn.microsoft.com/azure/virtual-machines/user-data)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
- [HackTricks Cloud: Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
