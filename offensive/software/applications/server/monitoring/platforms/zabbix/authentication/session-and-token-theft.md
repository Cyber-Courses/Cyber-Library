---
title: "Session and token theft: replaying stolen Zabbix sessions"
order: 6
description: "A Zabbix session is a zbx_sessionid cookie, and newer versions add long-lived API tokens; either authenticates without a password. Stolen from a browser, captured in transit over plain HTTP, or read from the sessions table in the Zabbix database, a session or token is replayed directly to assume the user's access, bypassing the login entirely."
keywords:
  - zbx_sessionid
  - api token
  - session theft
  - cookie
  - zabbix database
---

# Session and token theft

Authentication state in Zabbix is a bearer credential, so stealing it bypasses the login. The web session is a `zbx_sessionid` cookie; presenting it is being that user for the session's lifetime. Newer Zabbix versions add API tokens, which are long-lived strings created per user and sent as a bearer credential, convenient for automation and durable if stolen. Either is obtained several ways: lifted from a logged-in administrator's browser (XSS, local access), captured in transit where the frontend runs over plain HTTP, or read directly from the `sessions` table in the Zabbix database (and tokens from the `token` table) if the attacker reaches the database. A stolen session or token is replayed straight at the API or UI to assume the owner's access without any password.

```bash
Z=https://<target>/zabbix/api_jsonrpc.php; H='Content-Type: application/json-rpc'
# replay a stolen web session cookie
curl -sk https://<target>/zabbix/zabbix.php?action=dashboard.view -b 'zbx_sessionid=<stolen>'
# replay a stolen API token (bearer) on the API
curl -sk $Z -H "$H" -H 'Authorization: Bearer <stolen-token>' \
  -d '{"jsonrpc":"2.0","method":"host.get","params":{},"id":1}'
# from DB access: sessions and tokens are stored in the Zabbix schema
#   SELECT sessionid, userid FROM sessions;   SELECT token, userid FROM token;
```

## Exploitation notes

- The session cookie and API token are bearer credentials: no password is needed to replay them, so the attack is capture-then-reuse within the credential's validity.
- Plain-HTTP Zabbix frontends leak the `zbx_sessionid` to a sniffer; an admin session captured that way is immediate admin access, so an unencrypted frontend is a direct exposure.
- Database access is a potent source: the `sessions` and `token` tables hold active sessions and tokens, so a SQL-injection or DB compromise yields reusable authentication (and SQLi is itself a Zabbix surface, see [SQL injection to RCE](../code-execution/sql-injection-to-rce.md)).
- API tokens are long-lived and survive password changes until revoked, making a stolen token durable; prefer it for persistence. A replayed admin session routes to [code execution](../code-execution/index.md).

## References

- [Zabbix: API tokens](https://www.zabbix.com/documentation/current/en/manual/web_interface/frontend_sections/users/api_tokens)
- [HackTricks: Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
