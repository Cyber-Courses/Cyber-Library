---
title: "Weak passwords: common and reused Zabbix credentials"
order: 4
description: "Where the Admin password is changed, Zabbix has no strong password policy by default, so the replacement is often weak or reused: zabbix123, the company name, a season-and-year pattern, or a password reused from elsewhere. Combined with the enumerated user list, a short targeted spray frequently recovers a working account."
keywords:
  - weak password
  - zabbix123
  - password policy
  - reuse
  - spray
---

# Weak passwords

Older Zabbix versions enforce no password policy, so even when the default is changed the replacement is frequently weak: `zabbix`, `zabbix123`, `Password1`, the company or system name, or a season-and-year pattern, and administrators commonly reuse a password they use elsewhere. Combined with the [enumerated user list](../enumeration/user.md), this makes a short, targeted password spray effective: one likely password across the known accounts, focusing on the Super-admin users. The web login and the API both validate credentials, so the attack runs against either.

```bash
Z=https://<target>/zabbix/api_jsonrpc.php; H='Content-Type: application/json-rpc'
# spray one likely password across enumerated users via the API
for u in Admin zabbix monitoring operator; do
  r=$(curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"user.login","params":{"username":"'"$u"'","password":"zabbix123"},"id":1}')
  echo "$u: $r"; done
```

## Exploitation notes

- Spray one or a few context-derived passwords across the enumerated users rather than deep brute force; target Super-admin accounts first, as their access unlocks code execution.
- Newer Zabbix versions added a configurable password policy (and lockout), so feasibility depends on the [version](../enumeration/version-detection.md); older instances are unprotected.
- Credential reuse is common: a password found elsewhere in the environment may be the Zabbix admin's, and the Zabbix password may be reused for other services.
- A recovered account's value depends on its role; confirm with `user.get`/`role.get` and route to [code execution](../code-execution/index.md) if it is admin.

## References

- [Zabbix: password policy](https://www.zabbix.com/documentation/current/en/manual/web_interface/frontend_sections/users/authentication)
- [HackTricks: Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
