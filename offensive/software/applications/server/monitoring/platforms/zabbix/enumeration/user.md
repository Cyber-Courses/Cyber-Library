---
title: "User: enumerating Zabbix accounts"
description: "The user.get API method and the Users administration page list Zabbix accounts, their usernames, roles, and user groups. The list identifies the administrative accounts to target, confirms whether default accounts like Admin and guest exist, and feeds password spraying, since Zabbix accounts are a direct path to the platform's code-execution features."
keywords:
  - zabbix users
  - user.get
  - admin account
  - guest
  - role
---

# User

Enumerating Zabbix users identifies who to attack and how much each account is worth. The `user.get` API method (and the Administration > Users page) returns the accounts with their usernames, assigned roles, and user-group membership. This confirms whether the default `Admin` superadmin and the `guest` account exist, identifies the administrative accounts (Super admin role) that unlock code execution, and produces the username list for password spraying. Because a sufficiently privileged Zabbix account leads directly to running commands on the server and monitored hosts, knowing which accounts hold which roles focuses the credential attack on the highest-value targets.

```bash
Z=https://<target>/zabbix/api_jsonrpc.php; H='Content-Type: application/json-rpc'
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"user.get","params":{"output":["userid","username","roleid"],"selectUsrgrps":["name"]},"auth":"'"$TOK"'","id":1}'
# map roles to privileges (which accounts are Super admin)
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"role.get","params":{"output":"extend"},"auth":"'"$TOK"'","id":1}'
```

## Exploitation notes

- The role/usergroup mapping is the point: target Super-admin accounts for credential attacks, since they unlock [script execution](../code-execution/global-and-alert-scripts.md) and full configuration; a non-admin account may still enable item-based execution depending on permissions.
- Confirm the default accounts: a present `Admin` (default password `zabbix`) or an enabled `guest` is an immediate [default-credential](../authentication/default-credentials.md)/[guest-access](../authentication/guest-access.md) win.
- The username list feeds [brute force](../authentication/brute-force.md) and spraying; Zabbix logins are web-based, so respect any lockout and prefer spraying.
- `user.get` requires an authenticated session, so it follows initial access; before access, rely on default-account knowledge and the login page.

## References

- [Zabbix API: user.get](https://www.zabbix.com/documentation/current/en/manual/api/reference/user/get)
- [HackTricks: Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
