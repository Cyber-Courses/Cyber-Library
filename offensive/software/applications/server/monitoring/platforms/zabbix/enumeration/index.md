---
title: "Enumeration: mapping Zabbix through the API and UI"
order: 3
description: "Zabbix exposes a JSON-RPC API and a web UI that reveal the version (often unauthenticated), the user accounts, and the monitored hosts. The version drives exploit selection, the user list drives credential attacks, and the monitored-host inventory is a map of the internal estate with the IPs and services Zabbix reaches, all obtained through api_jsonrpc.php once authenticated or, for the version, before."
keywords:
  - zabbix enumeration
  - api_jsonrpc.php
  - apiinfo.version
  - host.get
  - user.get
---

# Enumeration

Zabbix enumeration works through the JSON-RPC API (`/api_jsonrpc.php`) and the web UI. The version is readable, usually without authentication, through `apiinfo.version` and the login-page footer, and it is the key fact for exploit selection. After authenticating (or via any session), the API enumerates the user accounts (`user.get`), which feeds credential attacks, and the monitored hosts (`host.get`), which is a map of the internal estate: the IP addresses, hostnames, and interfaces Zabbix reaches, plus the items and macros that often hold the credentials used to monitor them. The API is the efficient path; the UI shows the same data.

```bash
Z=https://<target>/zabbix/api_jsonrpc.php; H='Content-Type: application/json-rpc'
# version (no auth in many versions)
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"apiinfo.version","params":{},"id":1}'
# authenticate to get a token, then enumerate
TOK=$(curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"user.login","params":{"username":"Admin","password":"zabbix"},"id":1}' | jq -r .result)
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"user.get","params":{"output":["userid","username","roleid"]},"auth":"'"$TOK"'","id":1}'
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"host.get","params":{"output":["host"],"selectInterfaces":["ip","dns"]},"auth":"'"$TOK"'","id":1}'
```

## Subtopics

- **[Version detection](version-detection.md)**: reading the Zabbix version.
- **[API](api.md)**: the JSON-RPC API as the enumeration interface.
- **[User](user.md)**: enumerating accounts.
- **[Host](host.md)**: the monitored-host inventory.

## References

- [Zabbix API: apiinfo.version](https://www.zabbix.com/documentation/current/en/manual/api/reference/apiinfo/version)
- [HackTricks: Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
