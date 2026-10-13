---
title: "Guest access: the built-in read-only Zabbix account"
order: 3
description: "Zabbix has a built-in guest account that, when enabled, grants unauthenticated read access to whatever the guest user group is permitted to see, commonly dashboards and host data. Historically enabled by default, guest access exposes the monitored-host inventory and configuration without any credential, and over-broad guest permissions can widen that to sensitive data."
keywords:
  - guest access
  - zabbix guest
  - read-only
  - unauthenticated
  - dashboards
---

# Guest access

Zabbix includes a special `guest` user that provides access to unauthenticated visitors: when enabled, anyone reaching the frontend is treated as the guest user and sees whatever the guest's user group is permitted to see. Historically guest access was enabled by default, and it is still present where it has not been explicitly disabled. The exposure depends on the guest group's permissions: at minimum it typically reveals dashboards and the monitored-host list, which is a map of the internal estate; where an administrator granted the guest group broader read permissions (a common convenience), it exposes item data, problems, and configuration. All of this without any credential.

```bash
# simply browsing the frontend uses the guest session if enabled
curl -sk https://<target>/zabbix/zabbix.php?action=dashboard.view | grep -i dashboard
# the API confirms guest login (empty password) where enabled
curl -sk https://<target>/zabbix/api_jsonrpc.php -H 'Content-Type: application/json-rpc' \
  -d '{"jsonrpc":"2.0","method":"user.login","params":{"username":"guest","password":""},"id":1}'
# then enumerate what guest can see (hosts, dashboards)
```

## Exploitation notes

- Guest access is unauthenticated: reaching the frontend is the only requirement, so an exposed Zabbix with guest enabled leaks data to anyone.
- The value scales with the guest group's permissions: a default guest sees the host inventory (estate map), while an over-permissioned guest group exposes item values, problems, and configuration that may include sensitive detail.
- It is read-only, so it does not by itself give code execution, but the host/macro data it exposes is reconnaissance and sometimes credentials; combine with the enumeration pages.
- Check the [version](../enumeration/version-detection.md): guest was disabled by default in later releases, so its presence indicates an older or permissively-configured instance.

## References

- [Zabbix: guest user and permissions](https://www.zabbix.com/documentation/current/en/manual/web_interface/frontend_sections/users/user_groups)
- [HackTricks: Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
