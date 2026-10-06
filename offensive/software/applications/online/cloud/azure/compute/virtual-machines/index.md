---
title: "Virtual machines"
order: 1
description: "Taking over Azure VMs: run command and Custom Script Extension for SYSTEM or root execution, managed-identity token theft, and user-data and cloud-init secrets."
keywords:
  - virtual machines
  - run command
  - custom script extension
  - managed identity
  - user data
---

# Virtual machines

A VM you hold write rights over is both a code-execution target and a credential source. The management plane gives you execution without any stored credential (run command, an extension), and once you are on the box, the VM's attached **managed identity** hands out a subscription token from the metadata endpoint.

## Pages

- **[Run command](run-command.md)**: `runCommand` for SYSTEM or root execution with no credential.
- **[Custom Script Extension](custom-script-extension.md)**: `extensions/write` to run code, including VMAccess to reset the admin password.
- **[Managed identity](managed-identity.md)**: steal the VM identity's token from IMDS.
- **[User data](user-data.md)**: read cloud-init and user data for secrets, or write it to run at boot.

## References

- [HackTricks Cloud: Azure VMs](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: Azure privilege escalation via VM run command](https://www.netspi.com/blog/technical-blog/)
- [Microsoft: run command](https://learn.microsoft.com/azure/virtual-machines/run-command-overview)
