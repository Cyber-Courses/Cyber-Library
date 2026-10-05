---
title: "App Service"
description: "Attacking Azure App Service web apps: the Kudu and SCM console, deployment credentials, and the app's managed identity."
keywords:
  - App Service
  - Kudu
  - SCM
  - deployment credentials
  - managed identity
---

# App Service

An App Service web app exposes the **Kudu/SCM** management site (a web console and file API), ships with **publishing credentials** that push code, and can carry a **managed identity**. Any of the three gives code execution or an identity token: Kudu is a shell, publishing credentials deploy a web shell, and the managed identity is the prize behind both.

## What folds in here

- **[Kudu and SCM](kudu-and-scm.md)**: the management console for a web shell, file access, and environment secrets.
- **[Deployment credentials](deployment-credentials.md)**: publishing profiles that push code and read config.
- **[Managed identity](managed-identity.md)**: minting the app's managed-identity token.

## References

- [HackTricks Cloud: Azure App Service](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: Kudu service overview](https://learn.microsoft.com/azure/app-service/resources-kudu)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
