---
title: "Functions"
description: "Abusing Azure Functions: running as the app's managed identity and recovering function and host keys."
keywords:
  - Azure Functions
  - managed identity
  - function keys
  - host key
  - serverless
---

# Functions

An Azure Function app runs your code and, when one is configured, carries a **managed identity**. Code execution in the app (through a deploy, a dependency, or an injection) mints that identity's token; separately, the app's **function and host keys** authorize invoking protected functions and the admin endpoints.

## What folds in here

- **[Managed identity](managed-identity.md)**: minting and using the function app's managed-identity token.
- **[Function keys](function-keys.md)**: recovering host and function keys to invoke protected endpoints.

## References

- [HackTricks Cloud: Azure Functions](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: Azure Functions access keys](https://learn.microsoft.com/azure/azure-functions/function-keys-how-to)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
