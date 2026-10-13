---
title: "Authentication: gaining Zabbix access"
order: 1
description: "Zabbix access is gained through the web login or the API: the default Admin/zabbix credentials, weak passwords and brute force, an enabled guest account, API authentication-bypass flaws, and reuse of stolen session cookies or API tokens. Any of these yields a session whose privilege, especially Super admin, determines the path to code execution."
keywords:
  - zabbix authentication
  - default credentials
  - guest
  - api token
  - session
---

# Authentication

Gaining access to Zabbix is the gateway to its code-execution features and stored credentials, and there are several routes. The default `Admin` superadmin account ships with the password `zabbix` and is frequently unchanged. Weak passwords and the absence of a strong policy make brute force and spraying effective. The built-in `guest` account, where enabled, grants unauthenticated read access. Specific versions have had API authentication-bypass flaws. And stolen session cookies or API tokens are replayed directly. What matters after access is the account's role: a Super admin unlocks script execution and full configuration, while lesser roles may still enable item-based execution depending on permissions.

```bash
# web login and API login both validate the same credentials
Z=https://<target>/zabbix/api_jsonrpc.php
curl -sk $Z -H 'Content-Type: application/json-rpc' \
  -d '{"jsonrpc":"2.0","method":"user.login","params":{"username":"Admin","password":"zabbix"},"id":1}'
```

## Subtopics

- **[Default credentials](default-credentials.md)**: Admin/zabbix and guest.
- **[Weak passwords](weak-passwords.md)**: common and reused passwords.
- **[Brute force](brute-force.md)**: online attacks on the login.
- **[Guest access](guest-access.md)**: the built-in read-only guest account.
- **[API authentication bypass](api-authentication-bypass.md)**: version-specific auth-bypass flaws.
- **[Session and token theft](session-and-token-theft.md)**: replaying stolen sessions and API tokens.

## References

- [Zabbix authentication documentation](https://www.zabbix.com/documentation/current/en/manual/web_interface/frontend_sections/users)
- [HackTricks: Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
