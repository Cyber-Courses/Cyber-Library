---
title: "Configuration and exposure: Zabbix deployment weaknesses"
order: 2
description: "Independent of a single exploit, Zabbix is often deployed insecurely: the web frontend and API exposed to untrusted networks, and over-permissive roles and user groups that grant ordinary accounts the Super-admin or script-execution rights that lead to code execution. These configuration weaknesses widen who can reach the platform and what a given account can do."
keywords:
  - zabbix exposure
  - exposed frontend
  - roles
  - permissions
  - super admin
---

# Configuration and exposure

Beyond specific exploits, how Zabbix is deployed decides how exposed it is. Two configuration weaknesses recur. The frontend and API are frequently reachable from untrusted networks (the internet, or a flat internal network), which exposes the login, the version, and the guest/default-credential surface to anyone. And the role and user-group model is often too permissive: accounts are granted the Super-admin role or the script/item permissions that lead to code execution when they do not need them, so compromising an ordinary account yields far more than its purpose warranted. These are not single-shot exploits but the conditions that make the other attacks reachable and impactful.

## Subtopics

- **[Exposed web interface](exposed-web-interface.md)**: the frontend and API reachable from untrusted networks.
- **[Weak permissions](weak-permissions.md)**: over-privileged roles and user groups.

## References

- [Zabbix: user roles and permissions](https://www.zabbix.com/documentation/current/en/manual/web_interface/frontend_sections/users/user_roles)
- [HackTricks: Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
