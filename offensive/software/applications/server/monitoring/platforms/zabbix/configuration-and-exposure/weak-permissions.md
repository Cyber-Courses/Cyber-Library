---
title: "Weak permissions: over-privileged Zabbix roles and user groups"
description: "Zabbix controls access through roles and user-group permissions, and these are frequently too broad: ordinary accounts granted the Super-admin role, or user groups with write access to hosts and the script/item features. Over-privileged accounts turn a low-value credential into full configuration and code execution, so the effective privilege of any compromised account must be checked."
keywords:
  - weak permissions
  - super admin
  - user group
  - role
  - privilege
---

# Weak permissions

Zabbix authorizes actions through user roles (what UI/API features a user may use) and user-group permissions (which host groups a user may read or write). These are commonly configured too broadly: service and operator accounts are given the Super-admin role, or user groups are granted write access to hosts and the permission to create items and run scripts, far beyond what their purpose requires. The consequence for an attacker is leverage: compromising even a low-value account can yield full configuration and code execution if that account was over-privileged. So the effective privilege of any obtained account must be checked rather than assumed, and over-permissioned non-admin accounts are a quiet path to the same outcomes as the admin.

```bash
Z=https://<target>/zabbix/api_jsonrpc.php; H='Content-Type: application/json-rpc'
# what can this account actually do? (role features + host-group permissions)
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"role.get","params":{"output":"extend"},"auth":"'"$TOK"'","id":1}'
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"usergroup.get","params":{"output":"extend","selectRights":"extend"},"auth":"'"$TOK"'","id":1}'
# test the high-value capabilities directly: can you create a script or an item?
```

## Exploitation notes

- Check effective privilege, not the account name: an over-privileged operator or service account may have Super-admin or script/item-creation rights, which is the same as admin for reaching [code execution](../code-execution/index.md).
- The key capabilities to test are script management (server RCE via [scripts](../code-execution/global-and-alert-scripts.md)) and item write on hosts (agent RCE via [items](../code-execution/item-command-execution.md)); `role.get`/`usergroup.get` reveal whether the account holds them.
- Over-broad host-group write access also lets an attacker add hosts and interfaces pointing at systems they control, or reconfigure monitoring to harvest more credentials.
- This weakness makes lower-value credentials (easier to obtain) as useful as the admin; prioritise enumerating each obtained account's real rights.

## References

- [Zabbix: user roles](https://www.zabbix.com/documentation/current/en/manual/web_interface/frontend_sections/users/user_roles)
- [Zabbix: permissions](https://www.zabbix.com/documentation/current/en/manual/config/users_and_usergroups/permissions)
