---
title: "App settings and connection strings: looting Function and Web App config"
description: "Harvesting app settings, connection strings, and Kudu environment from Function and Web Apps for stored secrets and backend credentials."
keywords:
  - app settings
  - connection strings
  - Kudu
  - Function App
  - Web App
---

# App settings and connection strings

Function and Web Apps keep their configuration as **app settings** and **connection strings**: database passwords, storage keys, API tokens, and the Function host key. They are returned in plaintext to anyone with management access to the app, and are also readable from the Kudu/SCM environment of a running app.

## Reading from the management plane

```bash
az webapp config appsettings list --name <app> --resource-group <rg> -o json
az webapp config connection-string list --name <app> --resource-group <rg> -o json
az functionapp config appsettings list --name <fn> --resource-group <rg> -o json
```

## Reading from Kudu / the running app

With publishing access or code execution in the app, the same values are environment variables, reachable through the Kudu console or its VFS API:

```bash
# Kudu runs at https://<app>.scm.azurewebsites.net
curl -s -u '<deploy-user>:<deploy-pass>' \
  https://<app>.scm.azurewebsites.net/api/settings
# or run `env` in the Kudu debug console
```

## Exploitation notes

- Connection strings routinely contain storage account keys and SQL credentials, which pivot straight to [storage](../storage/index.md) and [data](../data/index.md).
- The `AzureWebJobsStorage` setting is the Function's own storage account key, full data-plane access to that account.
- Kudu publishing credentials themselves come from the app's [deployment credentials](../serverless/app-service/deployment-credentials.md); one feeds the other.

## Tools

- **az cli** (`webapp config`, `functionapp config`).
- **MicroBurst** (`Get-AzPasswords`): pulls app settings and publishing creds.
- **Kudu** console / `/api/settings`.

## References

- [Microsoft: configure app settings](https://learn.microsoft.com/azure/app-service/configure-common)
- [HackTricks Cloud: Azure App Service](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
