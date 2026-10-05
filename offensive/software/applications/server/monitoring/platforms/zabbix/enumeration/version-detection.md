---
title: "Version detection: reading the Zabbix version"
description: "The Zabbix version is disclosed without authentication through the API's apiinfo.version method and the login-page footer, and it is the fact that decides which exploits apply. Matching the version separates patched instances from those exposed to the known SQL-injection and remote-code-execution chains."
keywords:
  - zabbix version
  - apiinfo.version
  - footer
  - fingerprint
  - exploit matching
---

# Version detection

The Zabbix version is both easy to obtain and decisive, because the SQL-injection and remote-code-execution chains are version-specific. The API exposes it without authentication through `apiinfo.version` (one of the few unauthenticated methods), and the web UI shows it in the login-page and footer "Zabbix x.y.z" string. Reading it first tells you whether the instance is in range for a known pre-authentication exploit, and which authentication and code-execution behaviours to expect (defaults, API shape, and script handling changed across major versions).

```bash
Z=https://<target>/zabbix/api_jsonrpc.php
# unauthenticated version via the API
curl -sk $Z -H 'Content-Type: application/json-rpc' \
  -d '{"jsonrpc":"2.0","method":"apiinfo.version","params":{},"id":1}'
# version from the web UI footer/login page
curl -sk https://<target>/zabbix/index.php | grep -ioE 'Zabbix [0-9]+\.[0-9]+\.[0-9]+'
```

## Exploitation notes

- `apiinfo.version` answers without credentials, so the version is free; it maps directly to the applicable [known server exploits](../code-execution/known-server-exploits.md) and SQLi chains.
- Major versions differ in defaults and features (guest access, API tokens, script execution model), so the version also selects which authentication and code-execution techniques apply.
- The footer string corroborates the API result and works when the API path is filtered; both are pre-auth.
- Feed the version into exploit selection, then enumerate [users](user.md) and [hosts](host.md) once you have access.

## References

- [Zabbix API: apiinfo.version](https://www.zabbix.com/documentation/current/en/manual/api/reference/apiinfo/version)
- [HackTricks: Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
