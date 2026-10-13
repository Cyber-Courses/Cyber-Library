---
title: "Azure serverless"
order: 5
description: "Attacking Azure serverless and automation: Function and Logic App managed identities, Automation Account runbooks and RunAs, App Service Kudu and deployment, and Deployment Scripts."
keywords:
  - Azure serverless
  - Functions
  - Logic Apps
  - Automation Accounts
  - App Service
---

# Serverless

Azure's serverless and automation services all run code, and almost all of them run it under a **managed identity** or a stored credential, so each is a path from control of the service to the identity it carries. The pattern is the same across them: get code to execute (or a workflow to fire), then mint the service's token and continue as it.

Automation Accounts and App Service are the richest targets: Automation runbooks execute as a privileged identity and hold cleartext credential assets, and App Service exposes the Kudu console and publishing credentials.

## What folds in here

- **[Functions](functions/index.md)**: running as the function app's managed identity and recovering function and host keys.
- **[Logic Apps](logic-apps.md)**: adding a workflow step that calls ARM or Graph as the workflow's managed identity.
- **[Automation Accounts](automation-accounts/index.md)**: runbooks, the RunAs account, and hybrid workers for execution as a privileged identity.
- **[App Service](app-service/index.md)**: the Kudu and SCM console, deployment credentials, and the app's managed identity.
- **[Deployment Scripts](deployment-scripts.md)**: ARM `deploymentScripts` that run a container as a chosen user-assigned identity.

The managed-identity **token endpoint** itself is covered under [credentials](../credentials/instance-metadata/index.md); the privilege-escalation framing of attaching an identity is in [identity](../identity/privilege-escalation/index.md).

## References

- [HackTricks Cloud: Azure services](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
- [PowerZure](https://github.com/hausec/PowerZure)
