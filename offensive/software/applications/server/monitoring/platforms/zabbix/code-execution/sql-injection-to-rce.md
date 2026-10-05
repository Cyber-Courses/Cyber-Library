---
title: "SQL injection to RCE: Zabbix frontend injection chains"
description: "The Zabbix frontend has had SQL-injection vulnerabilities, some pre-authentication, in parameters that reach database queries unsafely. Beyond extracting data, a Zabbix SQLi recovers session IDs and credentials from the database to gain an authenticated (often admin) session, which then reaches code execution through the platform's script and item features."
keywords:
  - zabbix sql injection
  - jsrpc
  - pre-auth
  - session extraction
  - rce chain
---

# SQL injection to RCE

The Zabbix PHP frontend has shipped SQL-injection vulnerabilities, including pre-authentication ones, where a request parameter reaches a database query without proper parameterization. Directly, the injection extracts data from the Zabbix database: the `sessions` table (active session IDs), the `users` table (password hashes and, in some versions, secrets), the `token` table (API tokens), and the item/macro configuration that holds monitoring credentials. The chain to code execution is to use the injection to recover an administrator's session ID or credentials, log in as that admin, and then run commands through the [script](global-and-alert-scripts.md) or [item](item-command-execution.md) features. So a Zabbix SQLi, even an unauthenticated one, becomes full server code execution through the platform's own facilities.

```bash
# the vulnerable endpoint/parameter is version-specific (e.g. historic jsrpc.php
# and various frontend parameters); fingerprint the version and match the advisory.
# the chain, once injection is confirmed:
#  1. extract an active admin session id from the sessions table:
#       UNION SELECT sessionid FROM sessions JOIN users USING(userid) WHERE users.roleid=<superadmin>
#  2. replay that session (zbx_sessionid cookie) to become admin   (see session theft)
#  3. create+execute a global script for RCE on the Zabbix server  (see global/alert scripts)
```

## Exploitation notes

- The direct loot is the `sessions` and `token` tables: extracting an admin session ID or API token via the injection gives an authenticated admin context without cracking a password, replay it per [Session and token theft](../authentication/session-and-token-theft.md).
- The endpoint and parameter are version-specific; `apiinfo.version` gives the build to match the applicable SQLi advisory, and some are pre-authentication (no login needed to inject).
- From an admin session the RCE is the [global-script](global-and-alert-scripts.md) path on the Zabbix server; the SQLi is the access step, the scripts are the execution step.
- The same DB access also exposes the monitoring credentials in items/macros, so the injection is both an access and a credential-harvest primitive.

## References

- [Zabbix security advisories](https://support.zabbix.com/browse/ZBX)
- [HackTricks: Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
